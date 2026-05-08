---
layout: default
title: Códigos de programación
parent: "Proceso de manufactura"
nav_order: 3
---

# Códigos de programación

El sistema tiene **tres capas de software** independientes: el firmware del sensor, el backend de ML y la app móvil.

---

## 1. Firmware — ESP32-C3 (Arduino / C++)

### Entorno de desarrollo

- **IDE:** Arduino IDE 2.x
- **Placa:** ESP32-C3 (paquete `arduino-esp32` v2.x)
- **Lenguaje:** C++

### Librerías utilizadas

| Librería | Versión | Función |
|----------|---------|---------|
| `Wire.h` | built-in | Comunicación I2C con los sensores |
| `MadgwickAHRS.h` | arduino-MadgwickAHRS | Fusión de sensores → yaw/pitch/roll |
| `BLEDevice.h` | parte de arduino-esp32 | Stack BLE NUS (Nordic UART Service) |
| `BLE2902.h` | parte de arduino-esp32 | Descriptor CCCD para notificaciones BLE |

### Arquitectura del firmware

```
setup()
├── Inicializar BLE (nombre: ESP32_IMU_GOLF, UUID NUS)
├── Inicializar I2C (SDA=6, SCL=5, 100 kHz)
├── Inicializar ADXL345 (±2 g, 100 Hz)
├── Inicializar ITG3200 (±2000 °/s, 100 Hz)
├── Inicializar HMC5883L
├── calibrateGyro(500) — promedio de 500 muestras estáticas
└── Emitir encabezado CSV: "t_ms,ax,ay,az,gx,gy,gz,yaw,pitch,roll"

loop()  [~100 Hz]
├── readADXL345() → ax, ay, az [g]
├── readITG3200() → gx, gy, gz [°/s] − offsets
├── readHMC5883L() → mx, my, mz
├── filter.update(gx_rad, gy_rad, gz_rad, ax, ay, az, mx, my, mz)
├── roll = filter.getRoll()
│   pitch = filter.getPitch()
│   yaw = filter.getYaw()
├── snprintf → CSV line
├── bleNotifyLine() — en chunks de 20 bytes por BLE
└── delay(10)  → ~100 Hz
```

### Fragmento clave — transmisión BLE en chunks

La especificación BLE limita cada notificación a 20 bytes. Las líneas CSV pueden tener hasta 80 caracteres, por lo que se envían en fragmentos consecutivos:

```cpp
void bleNotifyLine(const String &msg) {
  String payload = msg + "\n";
  const int chunkSize = 20;
  int offset = 0;
  while (offset < payload.length()) {
    int len = min(chunkSize, (int)(payload.length() - offset));
    String chunk = payload.substring(offset, offset + len);
    pTxCharacteristic->setValue((uint8_t *)chunk.c_str(), len);
    pTxCharacteristic->notify();
    offset += len;
    delay(1);
  }
}
```

La app Android reassembla los chunks buscando el carácter `\n` que indica el fin de cada línea CSV.

### Fragmento clave — calibración de giroscopio

```cpp
void calibrateGyro(int samples = 500) {
  float sx = 0, sy = 0, sz = 0;
  for (int i = 0; i < samples; i++) {
    float gx, gy, gz;
    readITG3200(gx, gy, gz);
    sx += gx; sy += gy; sz += gz;
    delay(5);
  }
  gx_offset = sx / samples;
  gy_offset = sy / samples;
  gz_offset = sz / samples;
}
```

---

## 2. Pipeline de ML — Python

### Entorno de desarrollo

- **Lenguaje:** Python 3.11
- **Ejecución:** scripts independientes o importados por el backend

### Librerías utilizadas

| Librería | Función |
|----------|---------|
| `pandas` | Manejo de DataFrames de capturas CSV |
| `numpy` | Cómputo vectorizado de features |
| `scipy` | Filtro Gaussiano (`gaussian_filter1d`), estadísticas (`skew`) |
| `scikit-learn` | Random Forest, SVM, LOOCV |
| `matplotlib` | Gráficas EDA (boxplots, correlación, importancias) |
| `bleak` | Captura BLE desde PC (`capture_imu_ble.py`) |

### Arquitectura del pipeline

```
data/raw/*.csv          (buenos swings, label=1)
data/raw/Suboptimal_Swings/*.csv  (malos swings, label=0)
          │
          ▼
analyze_swing.py
  compute_signals()     → omega_mag, acc_mag, versiones suavizadas (Gaussian σ=2)
  find_swings()         → segmentación por umbral (inicio: omega > 60°/s, fin: omega < 30°/s)
          │
          ▼
extract_features.py
  extract_swing_features()  → dict con 19 features por swing
          │
          ▼
data/processed/swing_features.csv  (21 filas × 21 columnas)
          │
          ▼
train_classifier.py
  Leave-One-Out CV sobre 3 modelos: Rule-based, SVM (RBF), Random Forest
  → models/swing_classifier.pkl
          │
          ▼
classify_swing.py  /  api/services/pipeline_service.py
  predict_swing() → (label, confidence)
  feature_contributions() → importancias del RF por feature
```

### Las 19 features extraídas por swing

| Grupo | Features |
|-------|---------|
| Tempo | `tempo_ratio`, `T_backswing_s`, `T_downswing_s`, `T_total_s` |
| Velocidad angular | `omega_peak`, `omega_mean`, `omega_std`, `omega_skew`, `omega_at_top`, `omega_rise_rate`, `transition_dip_ratio` |
| Aceleración | `acc_peak`, `acc_mean`, `acc_std`, `acc_jerk_max` |
| Ángulos de Euler | `yaw_range`, `pitch_range`, `roll_range`, `yaw_at_top` |

### Segmentación de swings (fragmento)

```python
START_THRESHOLD = 60.0   # deg/s
END_THRESHOLD   = 30.0   # deg/s
MIN_SWING_SAMPLES = 25   # ~250 ms a 100 Hz

def find_swings(df):
    signal = df["omega_smooth"].values  # Gaussian σ=2
    swings = []
    in_swing = False
    for i, val in enumerate(signal):
        if not in_swing and val > START_THRESHOLD:
            in_swing = True; start_idx = i
        elif in_swing and val < END_THRESHOLD:
            if i - start_idx >= MIN_SWING_SAMPLES:
                swings.append((start_idx, i))
            in_swing = False
    return swings
```

### Rendimiento del clasificador (LOOCV, 21 swings)

| Modelo | Accuracy LOOCV | Recall "bad" |
|--------|---------------|--------------|
| Rule-based (umbrales acc_mean + omega_mean) | 61.9% | 80% |
| SVM (kernel RBF) | 71.4% | 0% |
| **Random Forest** | **76.2%** | **40%** |

Random Forest se seleccionó como modelo por defecto. Las 6 features más importantes: `acc_mean`, `omega_mean`, `yaw_at_top`, `acc_std`, `omega_at_top`, `T_backswing_s`.

---

## 3. Backend API — FastAPI (Python)

### Entorno de desarrollo

- **Lenguaje:** Python 3.11
- **Framework:** FastAPI 0.110+
- **Despliegue:** Docker + docker-compose
- **Base de datos:** SQLite vía `aiosqlite`

### Librerías de backend

| Librería | Función |
|----------|---------|
| `fastapi` | Framework HTTP async |
| `uvicorn` | Servidor ASGI |
| `aiosqlite` | SQLite asíncrono (historial de swings) |
| `pydantic` | Validación de schemas de request/response |
| `scikit-learn` | Carga del modelo `.pkl` |

### Endpoints de la API

| Método | Endpoint | Descripción |
|--------|----------|-------------|
| `GET` | `/health` | Estado del servidor, modelo y base de datos |
| `POST` | `/analyze` | Recibe lista de ImuSamples → segmenta → features → clasifica |
| `GET` | `/swings` | Historial paginado, filtrable por label |
| `GET` | `/swings/{id}` | Detalle completo de un swing |
| `POST` | `/swings/{id}/label` | Re-etiquetar un swing para reentrenamiento |
| `GET` | `/model/info` | Clase del modelo, features, tamaño del dataset |
| `POST` | `/model/retrain` | Re-corre `train_classifier.py` y recarga el modelo |

### Despliegue con Docker

```bash
# Construir imagen (~2-3 min, primera vez)
docker-compose build

# Arrancar en background
docker-compose up -d

# Verificar
curl http://localhost:8000/health
# → {"status":"ok","model_loaded":true,"db_connected":true}
```

Volúmenes persistentes:
- `./models` → `/app/models` (el modelo se actualiza al hacer retrain desde la app)
- `./api/swing_history.db` → `/app/api/swing_history.db` (historial de swings entre reinicios del contenedor)

---

## 4. App Android — Flutter (Dart)

### Entorno de desarrollo

- **Lenguaje:** Dart 3.3+
- **Framework:** Flutter 3.x
- **Plataforma:** Android ≥10

### Dependencias principales

| Paquete | Versión | Función |
|---------|---------|---------|
| `flutter_blue_plus` | ^1.31.15 | BLE: scan, connect, notificaciones |
| `flutter_riverpod` | ^2.5.1 | Gestión de estado reactiva |
| `dio` | ^5.4.3 | Cliente HTTP para el backend |
| `fl_chart` | ^0.68.0 | Gráfica en tiempo real de omega_mag |
| `go_router` | ^13.2.0 | Navegación declarativa |
| `shared_preferences` | ^2.2.3 | Almacenamiento local (URL del backend) |

### Pantallas de la app

| Pantalla | Función |
|----------|---------|
| **Home** | Resumen de sesión: barra good/bad, chips de swings recientes, botón "Start Session" |
| **Capture** | Estado de conexión BLE, gráfica en tiempo real de `omega_mag` + `acc_mag`, overlay de detección de swing, botón Start/Stop |
| **Result** | Banner GOOD/BAD, confianza %, timeline de fases (backswing/downswing/follow-through), desglose de features con barras de rango |
| **History** | Lista paginada de todos los swings anteriores, filtrable por Good/Bad |
| **Detail** | Desglose completo de features de cualquier swing histórico + botón para re-etiquetar |
| **Settings** | URL del backend, botón "Test Connection", umbrales de detección, info del modelo, botón Retrain |

### Arquitectura del software (Flutter)

```
main.dart
└── app.dart  (GoRouter, ProviderScope)
    ├── core/
    │   ├── ble_constants.dart   (SERVICE_UUID, TX_CHAR_UUID)
    │   ├── app_router.dart      (rutas: /, /capture, /result, /history, /settings)
    │   └── swing_detector.dart  (espejo en Dart del umbral omega: 60/30 °/s)
    ├── services/
    │   ├── ble_service.dart     (scan, connect, reassembly de chunks BLE → ImuSample)
    │   └── api/swing_api.dart   (POST /analyze, GET /swings, POST /model/retrain)
    ├── providers/
    │   ├── ble_provider.dart    (estado de conexión, stream de muestras)
    │   ├── capture_provider.dart (buffer de ImuSamples durante la sesión)
    │   └── history_provider.dart (cache del historial paginado)
    └── screens/
        ├── capture/live_chart.dart   (gráfica fl_chart de omega_mag en tiempo real)
        ├── result/verdict_banner.dart (GOOD/BAD con color + confianza)
        ├── result/phase_timeline.dart (timeline visual de las 3 fases)
        └── result/feature_breakdown_list.dart (lista de features con verde/rojo)
```

### Build y ejecución

```bash
cd golf_swing_analyzer
flutter pub get
flutter run -d <device-id>    # o: flutter run (detecta el dispositivo conectado)
```

---

## Pruebas de software

| Prueba | Descripción | Resultado |
|--------|-------------|-----------|
| Pipeline end-to-end | Capturar swing vía BLE → POST /analyze → veredicto en app | ✅ funciona en ≤2 s |
| Segmentación de swings | 13 archivos CSV → `extract_features.py` → 21 swings detectados | ✅ 0 falsos negativos en dataset |
| Clasificador LOOCV | `train_classifier.py` → Random Forest, 21 swings | ✅ 76.2% accuracy |
| Retrain desde app | Relabelar swing en Detail → tap Retrain en Settings → nuevo .pkl cargado | ✅ |
| BLE reconnection | Desconectar/reconectar teléfono durante sesión | ✅ el ESP32 reinicia advertising automáticamente |
| Tailscale remoto | App en red móvil → backend en PC en Ethernet vía Tailscale | ✅ latencia <200 ms |

---

## Repositorio del código

El código completo del proyecto está en el repositorio `GolfSwingAnalysis`:

```
GolfSwingAnalysis/
├── firmware/esp32_imu_capture/esp32_imu_capture.ino
├── scripts/   (pipeline ML: capture, analyze, extract, train, classify)
├── api/       (FastAPI backend)
├── golf_swing_analyzer/   (Flutter app)
├── data/      (raw CSVs + processed features + plots)
└── models/    (swing_classifier.pkl)
```

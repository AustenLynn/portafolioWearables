---
layout: default
title: Códigos de programación
parent: "Proceso de manufactura"
nav_order: 3
---

# Códigos de programación

Documentación del firmware, las librerías utilizadas y la lógica de control del wearable.

---

## Entorno de desarrollo

- **Firmware:** Arduino IDE 2.x con board `ESP32C3 Dev Module` (paquete Espressif ESP32 v3.x)
- **Backend:** Python 3.11, FastAPI, scikit-learn, pandas, numpy, scipy — ejecutado en Docker
- **App móvil:** Flutter 3.x (Dart), `flutter_blue_plus` para BLE, Riverpod para gestión de estado
- **Lenguajes:** C++ (firmware), Python (backend + ML pipeline), Dart (app)

---

## Librerías utilizadas

### Firmware (Arduino / C++)

| Librería | Versión | Función |
|----------|---------|---------|
| `Wire` | Built-in | Comunicación I2C con los sensores |
| `MadgwickAHRS` | 1.2.0 | Filtro de fusión sensorial (accel + gyro + mag → yaw/pitch/roll) |
| `BLEDevice` / `BLEServer` | Built-in (ESP32) | Servidor BLE con perfil NUS (Nordic UART Service) |
| `BLE2902` | Built-in (ESP32) | Descriptor CCCD para habilitar notificaciones BLE |

### Pipeline ML / Backend (Python)

| Librería | Versión | Función |
|----------|---------|---------|
| `pandas` | — | Lectura y manipulación de archivos CSV de swings |
| `numpy` | — | Operaciones vectorizadas sobre señales |
| `scipy` | — | Filtro Gaussiano 1D, estadísticas de distribución |
| `scikit-learn` | — | Random Forest, SVM, LOOCV, métricas de clasificación |
| `fastapi` | — | API REST para exponer el pipeline de análisis |
| `bleak` | — | Captura BLE desde PC (scripts de laboratorio) |

---

## Arquitectura del software

### Firmware (ESP32)

```
setup()
├── BLEDevice::init() + BLEServer + NUS Service
├── Wire.begin(SDA=6, SCL=5)
├── initADXL345() / initITG3200() / initHMC5883L()
├── filter.begin(100 Hz)
└── calibrateGyro(500 muestras)

loop() @ ~100 Hz
├── readADXL345(ax, ay, az)        → escala: 0.0039 g/LSB
├── readITG3200(gx, gy, gz)        → escala: 1/14.375 °/s/LSB, – offset
├── readHMC5883L(mx, my, mz)
├── filter.update(gx_rad, gy_rad, gz_rad, ax, ay, az, mx, my, mz)
├── roll/pitch/yaw = filter.getRoll/Pitch/Yaw()
└── bleNotifyLine("t_ms,ax,ay,az,gx,gy,gz,yaw,pitch,roll")
    └── chunked in 20-byte BLE notifications
```

### Backend Python

```
FastAPI app
├── POST /analyze
│   ├── pipeline_service.py
│   │   ├── compute_signals()        → omega_mag, acc_mag, Gaussian smooth
│   │   ├── find_swings()            → segmentación por umbral
│   │   └── extract_swing_features() → 19 features por swing
│   └── model_service.py (singleton)
│       └── RandomForest.predict_proba() → GOOD/BAD + confidence
├── GET  /swings                     → historial paginado (SQLite)
├── POST /swings/{id}/label          → relabeling para reentrenamiento
├── POST /model/retrain              → re-ejecuta train_classifier.py
└── GET  /health
```

---

## Código principal — Firmware

Fragmentos clave del archivo `firmware/esp32_imu_capture/esp32_imu_capture.ino`:

### Lectura de sensores I2C

```cpp
void readADXL345(float &ax, float &ay, float &az) {
  int16_t rawX = read16LE(ADXL345_ADDR, 0x32);
  int16_t rawY = read16LE(ADXL345_ADDR, 0x34);
  int16_t rawZ = read16LE(ADXL345_ADDR, 0x36);
  // Escala full-resolution ±2g: 3.9 mg/LSB
  ax = rawX * 0.0039f;
  ay = rawY * 0.0039f;
  az = rawZ * 0.0039f;
}

void readITG3200(float &gx, float &gy, float &gz) {
  int16_t rawX = read16BE(ITG3200_ADDR, 0x1D);
  int16_t rawY = read16BE(ITG3200_ADDR, 0x1F);
  int16_t rawZ = read16BE(ITG3200_ADDR, 0x21);
  // Sensibilidad: 14.375 LSB/(°/s), compensando offset de calibración
  gx = ((float)rawX / 14.375f) - gx_offset;
  gy = ((float)rawY / 14.375f) - gy_offset;
  gz = ((float)rawZ / 14.375f) - gz_offset;
}
```

### Calibración del giroscopio

```cpp
void calibrateGyro(int samples = 500) {
  printlnBoth("# Calibrando gyro. No muevas el sensor...");
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

### Transmisión BLE en chunks de 20 bytes

```cpp
void bleNotifyLine(const String &msg) {
  String payload = msg + "\n";
  const int chunkSize = 20;
  int offset = 0;
  while (offset < payload.length()) {
    int len = min((int)(payload.length() - offset), chunkSize);
    String chunk = payload.substring(offset, offset + len);
    pTxCharacteristic->setValue((uint8_t *)chunk.c_str(), len);
    pTxCharacteristic->notify();
    offset += len;
    delay(1);
  }
}
```

---

## Código principal — Pipeline Python

### Segmentación de swing (`analyze_swing.py`)

```python
START_THRESHOLD = 60.0   # deg/s
END_THRESHOLD   = 30.0   # deg/s
MIN_SWING_SAMPLES = 25   # ~250 ms a 100 Hz

def find_swings(df):
    signal = df["omega_smooth"].values  # Gaussiano σ=2
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

### Extracción de features (`extract_features.py`)

El pipeline extrae **19 features** por swing, agrupadas en cuatro categorías:

| Categoría | Features |
|-----------|---------|
| **Tempo** | `tempo_ratio`, `T_backswing_s`, `T_downswing_s`, `T_total_s` |
| **Velocidad angular** | `omega_peak`, `omega_mean`, `omega_std`, `omega_skew`, `omega_at_top`, `omega_rise_rate`, `transition_dip_ratio` |
| **Aceleración** | `acc_peak`, `acc_mean`, `acc_std`, `acc_jerk_max` |
| **Ángulos de Euler** | `yaw_range`, `pitch_range`, `roll_range`, `yaw_at_top` |

Las features más discriminantes (por Cohen's d con el dataset actual):

| Feature | Cohen's d | Interpretación |
|---------|-----------|----------------|
| `acc_mean` | 1.89 *** | Malos swings tienen aceleración media mucho más alta |
| `omega_mean` | 1.74 *** | Malos swings rotan ~2× más rápido en promedio |
| `yaw_at_top` | 1.67 *** | Giro de hombros insuficiente al tope del backswing |

### Entrenamiento del clasificador (`train_classifier.py`)

```python
SELECTED_FEATURES = [
    "acc_mean", "omega_mean", "yaw_at_top",
    "acc_std", "omega_at_top", "T_backswing_s",
]

rf_pipe = Pipeline([
    ("scaler", StandardScaler()),
    ("clf", RandomForestClassifier(
        n_estimators=200, max_depth=3,
        class_weight="balanced", random_state=42
    ))
])

# Validación con Leave-One-Out CV (única opción válida con n=21)
preds_rf = cross_val_predict(rf_pipe, X, y, cv=LeaveOneOut())
# → LOOCV Accuracy: 76.2%  |  Bad recall: 40%
```

---

## Pruebas de software

| Prueba | Descripción | Resultado |
|--------|-------------|-----------|
| Reconexión BLE | Desconectar el teléfono y reconectar → el ESP32 reinicia advertising automáticamente | ✅ |
| Segmentación | Un swing real en un archivo CSV de captura se detecta como un único segmento | ✅ |
| Waggle filtrado | Un movimiento corto (<250 ms) no se detecta como swing | ✅ |
| Latencia `/analyze` | POST con 400 muestras (~4 s de swing) → respuesta < 500 ms | ✅ |
| Persistencia SQLite | El historial de swings persiste después de `docker-compose down && up` | ✅ |
| Reentrenamiento | POST `/model/retrain` actualiza el modelo y el siguiente `/analyze` usa el nuevo modelo | ✅ |

---

## Repositorio del código

El código fuente completo está disponible en:

[https://github.com/AustenLynn/GolfSwingAnalysis](https://github.com/AustenLynn/GolfSwingAnalysis)

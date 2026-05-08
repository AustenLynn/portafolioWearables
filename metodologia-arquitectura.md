---
layout: default
title: Arquitectura electrónica
parent: "Metodología de diseño"
nav_order: 3
---

# Arquitectura electrónica

---

## Diagrama de bloques del sistema

```
┌──────────────────────────────────┐
│       SENSOR (en el guante)      │
│                                  │
│  ADXL345  ──┐                    │
│  ITG3200  ──┤─ I2C ─→ ESP32-C3  │
│  HMC5883L ──┘         │         │
│                   Madgwick       │
│               (fusión AHRS)      │
│                        │         │
│               BLE NUS TX        │
│               ~70 Hz CSV        │
│                        │         │
│            Batería LiPo 3.7V    │
│              → Cargador UNIT    │
└────────────────┬─────────────────┘
                 │ BLE (20-byte chunks)
                 ▼
┌──────────────────────────────────┐
│       APP ANDROID (Flutter)      │
│                                  │
│  BleService → reassembly CSV     │
│  SwingDetector (Dart, live)      │
│  Pantallas: Home / Capture /     │
│             Result / History /   │
│             Detail / Settings    │
│                                  │
│  POST /analyze → JSON payload    │
└────────────────┬─────────────────┘
                 │ HTTP (LAN WiFi o Tailscale)
                 ▼
┌──────────────────────────────────┐
│   BACKEND FASTAPI (Docker, PC)   │
│                                  │
│  POST /analyze                   │
│    → segmentación (omega_mag)    │
│    → 19 features extraídas       │
│    → Random Forest → GOOD/BAD   │
│                                  │
│  GET  /swings  (historial)       │
│  POST /swings/{id}/label         │
│  POST /model/retrain             │
│  GET  /health                    │
│                                  │
│  SQLite: swing_history.db        │
│  Model:  swing_classifier.pkl    │
└──────────────────────────────────┘
```

---

## Componentes principales

| Componente | Función | Modelo / Referencia |
|------------|---------|---------------------|
| Microcontrolador | Leer sensores I2C, calcular AHRS, transmitir por BLE | ESP32-C3 |
| Acelerómetro | Medir aceleración en 3 ejes (ax, ay, az) | ADXL345 — ±2 g, 100 Hz |
| Giroscopio | Medir velocidad angular en 3 ejes (gx, gy, gz) | ITG3200 — ±2000 deg/s, 100 Hz |
| Magnetómetro | Medir campo magnético (mx, my, mz) para referencia de yaw absoluta | HMC5883L |
| Filtro AHRS | Fusionar accel + gyro + mag para calcular yaw/pitch/roll | Madgwick filter (biblioteca Arduino) |
| Batería | Alimentación autónoma del sensor | LiPo 3.7 V (≈500 mAh) |
| Módulo de carga | Carga la LiPo por USB-C | UNIT Battery Charger (basado en IP5306) |
| Teléfono Android | Interfaz de usuario + cliente BLE + cliente HTTP | Android ≥10, cualquier marca |
| PC (servidor) | Ejecutar el backend FastAPI + modelo ML | Windows 10/11, Docker Desktop |
| Tailscale | Túnel VPN cifrado punto a punto entre PC y teléfono | Tailscale (gratuito, hasta 3 dispositivos) |

---

## Criterios de selección de componentes

**ESP32-C3:** Compatible con Arduino IDE, BLE nativo (soporte NUS out-of-the-box con la biblioteca `arduino-esp32`), bajo consumo, precio accesible y disponible en el lab. El ESP32-C3 soporta BLE 5.0 con conexión estable a ≥5 m, suficiente para uso en campo.

**ADXL345 + ITG3200 + HMC5883L:** Estos tres sensores forman el stack clásico "10 DOF IMU". Existen como módulo integrado de bajo costo, con comunicación I2C unificada en el mismo bus. El ADXL345 y el ITG3200 son los sensores activos de la pipeline; el HMC5883L aporta la referencia de heading para que el filtro Madgwick calcule yaw absoluta.

**Madgwick AHRS:** Frente a un complementary filter simple (gyro + accel), el filtro Madgwick converge más rápido al inicio de sesión y produce yaw estable gracias al magnetómetro. Disponible como biblioteca Arduino sin dependencias adicionales.

**Flutter (app):** Cross-platform con soporte BLE maduro (`flutter_blue_plus`) y manejo de estado reactivo (Riverpod). La app es Android-only en este prototipo pero el código puede compilarse para iOS sin cambios mayores.

**FastAPI + Docker:** FastAPI permite reutilizar los scripts Python del pipeline ML directamente como módulos importados, sin código duplicado. Docker garantiza que el servidor arranca en cualquier PC con el mismo entorno, sin conflictos de dependencias.

---

## Esquema de conexiones (pinout ESP32-C3)

| Pin ESP32-C3 | Señal | Componente |
|-------------|-------|------------|
| GPIO 5 | SCL | ADXL345, ITG3200, HMC5883L |
| GPIO 6 | SDA | ADXL345, ITG3200, HMC5883L |
| 3V3 | VCC | Todos los sensores |
| GND | GND | Todos los sensores |
| — | I2C addr ADXL345 | 0x53 |
| — | I2C addr ITG3200 | 0x68 |
| — | I2C addr HMC5883L | 0x1E |

**BLE UUIDs:**

| Parámetro | Valor |
|-----------|-------|
| Nombre del dispositivo | `ESP32_IMU_GOLF` |
| Service UUID | `6e400001-b5a3-f393-e0a9-e50e24dcca9e` |
| TX Characteristic (notify) | `6e400003-b5a3-f393-e0a9-e50e24dcca9e` |
| RX Characteristic (write) | `6e400002-b5a3-f393-e0a9-e50e24dcca9e` |

---

## Consideraciones de energía

- **Voltaje de operación del sensor:** 3.3 V (regulado por el ESP32-C3).
- **Corriente máxima estimada:** ~120 mA (ESP32 transmitiendo BLE + 3 sensores I2C activos).
- **Batería:** LiPo 3.7 V ≈500 mAh → autonomía estimada de **≥4 horas** de transmisión continua, suficiente para una sesión de práctica completa.
- **Carga:** USB-C a través del módulo UNIT Battery Charger; indicador LED verde cuando está cargando.

---

## Implementación del stack de sensores

![Stack de PCBs: cargador + sensor IMU](assets/img/evidencia/electronica-pcb-stack.jpg)
*Figura — Stack de tarjetas electrónicas: cargador UNIT (abajo) + módulo IMU 10 DOF (arriba). El stack completo mide aproximadamente 40 mm × 25 mm × 15 mm.*

![Componentes sobre la mesa](assets/img/evidencia/electronica-componentes-mesa.jpg)
*Figura — Componentes del sistema extendidos en el banco de trabajo antes del ensamblaje: cargador, batería LiPo, pines del sensor, motor vibrador y cableado.*

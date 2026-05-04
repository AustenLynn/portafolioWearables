---
layout: default
title: Arquitectura electrónica
parent: "Metodología de diseño"
nav_order: 3
---

# Arquitectura electrónica

Descripción del sistema electrónico del wearable: componentes, diagrama de bloques y criterios de selección.

---

## Diagrama de bloques

```
┌──────────────────────────────────────────────────┐
│                 Sensor Wearable (ESP32-C3)        │
│                                                  │
│  ADXL345 ──┐                                     │
│  (I2C 0x53)│                                     │
│             ├──► Madgwick AHRS Filter ──► BLE    │
│  ITG3200 ──┤      (yaw, pitch, roll)    NUS TX   │
│  (I2C 0x68)│                                     │
│             │                                    │
│  HMC5883L ─┘                                     │
│  (I2C 0x1E)                                      │
│                                                  │
│  SDA: GPIO6 / SCL: GPIO5                         │
└──────────────────────────────────────────────────┘
                        │
               BLE 4.2 (NUS profile)
               ~70 Hz CSV, chunks 20 B
                        │
                        ▼
┌──────────────────────────────────────────────────┐
│               Android App (Flutter)              │
│  - Reassembla chunks → líneas CSV completas      │
│  - Detección de swing en tiempo real (Dart)      │
│  - POST /analyze → FastAPI Backend               │
└──────────────────────────────────────────────────┘
                        │
           HTTP (WiFi LAN o Tailscale VPN)
                        │
                        ▼
┌──────────────────────────────────────────────────┐
│     FastAPI Backend Python (Docker, PC)          │
│  - Segmentación de swing (omega_mag threshold)   │
│  - Extracción de 19 features biomecánicas        │
│  - Clasificación con Random Forest               │
│  - Persistencia en SQLite                        │
└──────────────────────────────────────────────────┘
```

---

## Componentes principales

| Componente | Función | Modelo / Referencia |
|------------|---------|---------------------|
| Microcontrolador | Leer sensores I2C, fusionar con Madgwick, transmitir CSV por BLE | ESP32-C3 (Espressif) |
| Acelerómetro | Medir aceleración 3-ejes a ±2 g, 100 Hz | ADXL345 (Analog Devices) |
| Giroscopio | Medir velocidad angular 3-ejes a ±2000 °/s, 100 Hz | ITG3200 (InvenSense) |
| Magnetómetro | Medir campo magnético para referencia de yaw absoluto | HMC5883L (Honeywell) |
| Filtro de fusión | Calcular yaw/pitch/roll a partir de los tres sensores | Madgwick AHRS (Arduino lib) |
| BLE module | Transmitir datos al teléfono (perfil NUS — Nordic UART Service) | Integrado en ESP32-C3 |
| Fuente de energía | Alimentación portátil durante el swing | LiPo 3.7 V 500 mAh + módulo TP4056 |

---

## Criterios de selección de componentes

- **ESP32-C3:** BLE 5.0 integrado, RISC-V 160 MHz, footprint pequeño (~18 mm × 20 mm), programable con Arduino IDE, costo < 8 USD. No requiere módulo BLE externo.
- **ADXL345:** Rango ±2 g con resolución de 3.9 mg/LSB en modo full resolution — suficiente para capturar el impacto del swing (hasta ~3 g). Comunicación I2C hasta 400 kHz.
- **ITG3200:** Rango ±2000 °/s — el swing de golf genera picos de ~400–900 °/s medidos en el dataset. Sensibilidad de 14.375 LSB/°/s.
- **HMC5883L:** Permite referenciar el yaw respecto al campo magnético terrestre, lo que da `yaw_at_top` (rotación de hombros absoluta) como feature discriminante (Cohen's d = 1.67).
- **Madgwick AHRS:** Eficiente computacionalmente (corre en ESP32 sin pérdidas de muestras), bajo drift de yaw comparado con integración pura de giroscopio.

---

## Esquema de conexiones (pinout)

| Señal | Pin ESP32-C3 | Sensor |
|-------|-------------|--------|
| SDA | GPIO 6 | ADXL345 SDA, ITG3200 SDA, HMC5883L SDA |
| SCL | GPIO 5 | ADXL345 SCL, ITG3200 SCL, HMC5883L SCL |
| VCC | 3.3 V | Todos los sensores |
| GND | GND | Todos los sensores |

Los tres sensores comparten el bus I2C con direcciones distintas:
- ADXL345: `0x53` (SDO en HIGH)
- ITG3200: `0x68` (AD0 en LOW)
- HMC5883L: `0x1E` (fija)

---

## Consideraciones de energía

- **Voltaje de operación:** 3.3 V (regulado por el módulo ESP32-C3; la batería LiPo entrega 3.7 V → 4.2 V)
- **Corriente máxima estimada:**
  - ESP32-C3 activo + BLE: ~160 mA
  - ADXL345 + ITG3200 + HMC5883L: ~8 mA total
  - **Total: ~170 mA**
- **Tipo de batería / fuente:** LiPo 3.7 V 500 mAh con módulo de carga TP4056
- **Autonomía estimada:** 500 mAh / 170 mA ≈ **2.9 horas** de operación continua (suficiente para una sesión de práctica)

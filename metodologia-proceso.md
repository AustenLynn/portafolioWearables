---
layout: default
title: Proceso de diseño
parent: "Metodología de diseño"
nav_order: 1
---

# Proceso de diseño

Descripción de la **metodología** adoptada y las etapas que se siguieron para desarrollar el wearable.

---

## Metodología empleada

Se adoptó una metodología de **Prototipado Iterativo** con ciclos cortos de construcción-medición-aprendizaje, similar a Lean UX. Dada la naturaleza técnica del proyecto (hardware + firmware + ML + app móvil), el proceso fue altamente iterativo: cada subsistema se validó de forma independiente antes de integrarse con el siguiente, y los parámetros de segmentación y clasificación se ajustaron conforme se acumularon más datos de swing.

---

## Etapas del proceso

### Etapa 1 — Investigación y empatía

Se revisó literatura académica y productos comerciales sobre análisis de swing con IMU. Se identificaron los parámetros biomecánicos más relevantes para la calidad del golpe de golf:

- **Tempo** (relación tiempo-backswing / tiempo-downswing, objetivo ~3:1)
- **Velocidad angular máxima** y su timing (rápido en el downswing = bueno)
- **Rotación de hombros** (yaw al tope del backswing ≥ 269°)
- **Aceleración media** en el swing (alta = movimiento descontrolado)

También se evaluaron plataformas de microcontroladores: ESP32-C3 fue seleccionado por su tamaño compacto, soporte nativo BLE y facilidad de programación con Arduino.

### Etapa 2 — Definición del problema

**Problem statement:** *"Un golfista aficionado practicando en solitario no puede saber si su swing tiene defectos técnicos sin retroalimentación externa. El sistema debe detectar y clasificar automáticamente cada swing usando un sensor IMU wearable, sin requerir cámaras ni asistencia humana."*

Restricciones clave identificadas:
- El sensor no debe interferir con el agarre natural del palo.
- La latencia desde el fin del swing hasta el resultado debe ser < 5 s.
- El modelo debe funcionar con datasets pequeños (< 50 swings por clase).

### Etapa 3 — Ideación

Se generaron bocetos de tres posibles arquitecturas de sistema:

1. **Arquitectura local** — todo en el teléfono (descartada: inferencia en tiempo real con scikit-learn en Android era compleja).
2. **Arquitectura cloud** — backend en servidor remoto (descartada: latencia y dependencia de red en el campo).
3. **Arquitectura LAN/Tailscale** — backend Python en PC del usuario, acceso por WiFi o túnel Tailscale ✅ (elegida).

Para el sensor, se decidió usar tres chips separados (ADXL345 + ITG3200 + HMC5883L) en lugar de un IMU integrado (p. ej. MPU-6050) para obtener mayor resolución en cada eje y poder usar el filtro de fusión Madgwick completo con magnetómetro.

### Etapa 4 — Prototipado

El proceso de prototipado siguió cuatro fases paralelas:

| Fase | Subsistema | Resultado |
|------|-----------|-----------|
| P1 | Firmware ESP32 + lectura de sensores vía I2C | CSV correcto por Serial/USB |
| P2 | BLE NUS con chunks de 20 bytes | App Android recibe líneas completas |
| P3 | Pipeline Python: segmentación + extracción de features | swing_features.csv con 21 swings |
| P4 | Clasificador Random Forest + API FastAPI + Docker | `/analyze` devuelve veredicto en ~800 ms |

### Etapa 5 — Evaluación e iteración

- **Iteración 1 (firmware):** La calibración del giroscopio se hacía solo en `setup()`. Problema: el ESP32 tardaba ~3 s en inicializar BLE antes de calibrar, causando desfase. Solución: mover calibración después de `BLEDevice::startAdvertising()` para que el cliente pueda conectarse durante la espera.
- **Iteración 2 (segmentación):** El umbral `START_THRESHOLD = 100 deg/s` perdía el inicio del backswing. Se redujo a `60 deg/s` y se agregó suavizado Gaussiano (σ=2) para eliminar falsos positivos de movimientos bruscos.
- **Iteración 3 (modelo):** SVM (kernel RBF) logró 71.4% en LOOCV pero recall de clase Bad = 0% (el hiperparámetro `C` colapsa en escalas pequeñas). Random Forest con `class_weight="balanced"` obtuvo 76.2% y Bad recall = 40%, siendo preferible dada la asimetría del dataset.

---

## Cronograma

| Semana | Actividad |
|--------|-----------|
| 1 | Selección de componentes, lectura de datasheets, montaje en breadboard |
| 2 | Firmware: lectura de sensores I2C, calibración de giroscopio, filter Madgwick |
| 3 | Firmware: BLE NUS con chunks de 20 bytes; script `capture_imu_ble.py` |
| 4 | Captura de swings etiquetados (8 buenos, 5 malos); `analyze_swing.py` |
| 5 | Extracción de features (19 features); exploración con Cohen's d |
| 6 | Entrenamiento de clasificadores; API FastAPI con Docker |
| 7 | App Flutter: BLE + chart en tiempo real + pantalla de resultado |
| 8 | Integración completa; pruebas en campo; reentrenamiento desde la app |

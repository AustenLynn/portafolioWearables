---
layout: default
title: Proceso de manufactura
nav_order: 30
has_children: true
---

# Proceso de manufactura

Esta sección documenta **cómo se fabricó** el wearable: desde los procesos físicos de construcción hasta la implementación del código que lo hace funcionar.

---

## Contenido de esta sección

| Subsección | Descripción |
|---|---|
| [Procesos de fabricación](manufactura-fabricacion.md) | Técnicas y pasos de construcción física |
| [Implementación electrónica](manufactura-electronica.md) | Montaje del circuito y pruebas de hardware |
| [Códigos de programación](manufactura-codigo.md) | Firmware, librerías y lógica de control |

---

## Resumen del proceso

El flujo de fabricación siguió un orden de dependencias naturales:

1. **Electrónica primero:** Se montaron y probaron individualmente los sensores (ADXL345, ITG3200, HMC5883L) en una breadboard con el ESP32-C3, validando la lectura I2C por puerto Serial antes de agregar BLE.
2. **Firmware después:** Una vez confirmada la lectura correcta de los tres sensores, se integró el filtro Madgwick y la transmisión BLE por chunks de 20 bytes.
3. **Software y ML:** Con el firmware funcional, se capturaron swings reales, se desarrolló el pipeline Python de segmentación + extracción de features + entrenamiento del clasificador.
4. **API y app al final:** El backend FastAPI se encapsuló en Docker, y la app Flutter se construyó para consumir la API y mostrar los resultados en tiempo real.

Cada fase fue validada de forma independiente antes de avanzar a la siguiente, minimizando el riesgo de bugs de integración.

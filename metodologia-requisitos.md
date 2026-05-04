---
layout: default
title: Requisitos técnicos
parent: "Metodología de diseño"
nav_order: 2
---

# Requisitos técnicos

Especificaciones funcionales, de rendimiento y de restricción que el wearable debe cumplir.

---

## Requisitos funcionales

| # | Requisito | Prioridad |
|---|-----------|-----------|
| RF-01 | El sensor debe capturar aceleración, velocidad angular y orientación (yaw/pitch/roll) de forma continua durante el swing | Alta |
| RF-02 | Los datos deben transmitirse inalámbricamente al teléfono sin cable ni interrupción del agarre | Alta |
| RF-03 | El sistema debe detectar automáticamente el inicio y fin de cada swing sin interacción del usuario | Alta |
| RF-04 | Cada swing debe clasificarse como BUENO o MALO con un porcentaje de confianza | Alta |
| RF-05 | El resultado debe aparecer en el teléfono en menos de 5 segundos desde el fin del swing | Alta |
| RF-06 | El usuario debe poder etiquetar (relabelar) swings y reentrenar el modelo desde la app | Media |
| RF-07 | El historial de swings debe persistir entre sesiones | Media |
| RF-08 | La app debe mostrar una gráfica en tiempo real de la magnitud de velocidad angular (`omega_mag`) durante la captura | Baja |

---

## Requisitos no funcionales

| # | Requisito | Descripción |
|---|-----------|-------------|
| RNF-01 | Frecuencia de muestreo | ≥ 70 Hz (se logró ~100 Hz con el loop de `delay(10)` en el firmware) |
| RNF-02 | Latencia BLE | < 50 ms por muestra; transmisión en chunks de 20 bytes por limitación de BLE |
| RNF-03 | Tamaño del sensor | Compatible con el grip de un palo de golf estándar (≤ 5 cm × 3 cm) |
| RNF-04 | Autonomía | ≥ 1 hora de captura continua con batería LiPo de 500 mAh |
| RNF-05 | Resistencia mecánica | El sensor debe soportar la aceleración de impacto (hasta ~3.14 g medidos en malos swings) |
| RNF-06 | Compatibilidad | Android 8.0+ con BLE 4.0; backend en Windows/Linux con Python 3.10+ |
| RNF-07 | Seguridad de datos | Los swings se almacenan localmente en SQLite, sin envío a servidores externos |

---

## Restricciones de diseño

- **Presupuesto:** Hardware < 30 USD (ESP32-C3 ~8 USD, sensores ~6 USD total, batería ~4 USD).
- **Materiales disponibles:** Componentes discretos (no PCB personalizado en prototipo V1).
- **Tiempo de fabricación:** 8 semanas, una persona.
- **Conexión única BLE:** El ESP32-C3 acepta solo un cliente BLE simultáneo; no se puede tener el script Python de captura y la app Android conectados al mismo tiempo.
- **Dataset pequeño:** El clasificador debe funcionar con LOOCV con ≥ 15 swings; no es viable usar redes neuronales profundas.

---

## Criterios de éxito

| Criterio | Métrica | Umbral mínimo |
|----------|---------|---------------|
| Detección de swing | % de swings reales capturados automáticamente | ≥ 90% |
| Precisión del clasificador | LOOCV Accuracy con Random Forest | ≥ 70% |
| Latencia extremo-a-extremo | Tiempo desde fin de swing hasta veredicto en pantalla | ≤ 5 s |
| Frecuencia de muestreo real | Hz medidos en las capturas | ≥ 70 Hz |
| Persistencia de datos | Swings guardados correctamente en SQLite | Sin pérdidas después de reinicio del backend |

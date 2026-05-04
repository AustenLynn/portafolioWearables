---
layout: default
title: Fotografías
parent: "Evidencia"
nav_order: 1
---

# Fotografías

Registro visual del proceso de diseño, fabricación y resultado final del wearable Golf Swing Analyzer.

---

## Proceso de diseño

![Bocetos de arquitectura del sistema](assets/img/evidencia-diseno-01.png)
*Figura 1 — Bocetos iniciales de la arquitectura ESP32 + sensores + BLE + app Android.*

---

## Fabricación

![Montaje en protoboard](assets/img/evidencia-fabricacion-01.png)
*Figura 2 — Módulos ADXL345, ITG3200 y HMC5883L montados en protoboard junto al ESP32-C3. Se muestran las conexiones I2C compartidas (SDA: GPIO6, SCL: GPIO5).*

![Integración con batería LiPo](assets/img/evidencia-fabricacion-02.png)
*Figura 3 — Conjunto electrónico con batería LiPo 500 mAh y módulo de carga TP4056 antes de empacar en la funda.*

---

## Implementación electrónica

![Prueba de señal por Serial Monitor](assets/img/evidencia-electronica-01.png)
*Figura 4 — Captura del Serial Monitor de Arduino IDE mostrando las columnas CSV: `t_ms, ax, ay, az, gx, gy, gz, yaw, pitch, roll` a ~100 Hz.*

![App Python capturando por BLE](assets/img/evidencia-electronica-02.png)
*Figura 5 — Script `capture_imu_ble.py` recibiendo líneas CSV del ESP32 a través de BLE y guardando en `data/raw/`.*

---

## Producto final

![Sensor montado en el palo de golf](assets/img/evidencia-final-frontal.png)
*Figura 6 — El módulo wearable fijado con velcro en la empuñadura del palo de golf. El conjunto mide ~9 × 5 × 2 cm y pesa ~45 g.*

![App Android — pantalla de resultado](assets/img/evidencia-final-uso.png)
*Figura 7 — Pantalla de resultado en la app Flutter mostrando: veredicto BUENO/MALO, porcentaje de confianza y desglose de las 6 features principales del swing analizado.*

![Historial de swings en la app](assets/img/evidencia-final-historial.png)
*Figura 8 — Pantalla de historial con la lista paginada de swings anteriores, filtrable por BUENO/MALO.*

---

> **Nota:** Las imágenes se agregarán conforme se complete la documentación fotográfica del proyecto. Guarda las fotos en `assets/img/` con los nombres de archivo indicados arriba.

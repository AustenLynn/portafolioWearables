---
layout: default
title: Conclusiones
parent: "Pulseras"
nav_order: 5
---

# Conclusiones — Pulseras

---

## Reflexión sobre el proceso

Ambas pulseras fueron proyectos de ciclo corto: una sola versión, sin iteraciones formales. Eso significa que el aprendizaje salió principalmente de los problemas encontrados durante la integración, no de ciclos de refinamiento.

**Lo que funcionó bien:**

Validar el circuito en protoboard antes de tocar el sustrato fue la decisión correcta en ambos casos. Cada vez que se intentó integrar algo que no había sido validado eléctricamente primero, el resultado fue un circuito roto y tiempo perdido desbordando costura.

En la Pulsera 2, la decisión de calibrar empíricamente los umbrales del micrófono (en lugar de asumir el rango teórico 0–1023) fue crítica: el micrófono en condiciones reales de laboratorio tiene un rango útil mucho más estrecho. Sin esa calibración, la tira habría estado casi siempre en máxima intensidad.

**Lo que cambiaría:**

En la Pulsera 1, se habría diseñado la banda desde el inicio pensando en la curvatura. Las pistas de cobre pegadas en plano funcionan sobre una superficie rígida pero fallan al doblar. Una solución más robusta sería usar hilo conductor cosido en lugar de cinta de cobre, ya que el hilo tolera la flexión sin romperse.

En la Pulsera 2, el Arduino quedó externo porque es demasiado grande. Con una segunda versión se buscaría un microcontrolador más pequeño — XIAO SAMD21 o ESP32-C3 en módulo compacto — que pudiera ir cosido directamente a la pulsera y hacer el sistema verdaderamente autónomo.

---

## Reflexión sobre el resultado

### ¿Las pulseras logran sus objetivos?

**Pulsera 1:**

| Objetivo | ¿Cumplido? | Observación |
|----------|-----------|-------------|
| 5 LEDs encendidos con conductores integrados | ✅ | Funcional en banco; la cinta de cobre se despega bajo flexión repetida |
| Estética coherente con el concepto wearable | ⚠️ | El resultado es funcional pero los conductores son visibles y la construcción es frágil |

**Pulsera 2:**

| Objetivo | ¿Cumplido? | Observación |
|----------|-----------|-------------|
| NeoPixel reactiva al sonido en tiempo real | ✅ | Latencia <100 ms, variación visible y sincronizada |
| Integración de electrónica en sustrato wearable | ⚠️ | Solo la tira LED está integrada; el Arduino queda externo |
| Sistema autónomo (sin cables a PC) | ❌ | El Arduino requiere USB o batería externa no integrada al sustrato |

---

## Aprendizajes clave

**La integración de electrónica en textil no es solo un problema de adhesivo.** La diferencia entre "pegar un módulo al guante" y "integrar electrónica en un wearable" es mecánica: los conductores deben tolerar los mismos esfuerzos que el sustrato. Cinta de cobre sobre foam es una solución de prototipado rápido, no una solución de producto.

**El tamaño del microcontrolador define la viabilidad del wearable.** El Arduino MKR WiFi (61.5 × 25 mm) es demasiado grande para la mayoría de pulseras. El ESP32-C3 del proyecto Golf Sync (25 × 20 mm con el módulo IMU) ya era un límite: en pulseras, el espacio disponible es aún más restrictivo. Los wearables de muñeca reales usan chips desnudos (bare die) o módulos de 10 × 10 mm máximo.

**La calibración empírica del sensor es parte del diseño.** El micrófono analógico no entrega una señal limpia y predecible — el ruido de fondo del laboratorio, la distancia al sonido y la ganancia del módulo determinan el rango útil real. Mapear el rango teórico sin calibración produce un sistema que no responde o que siempre está saturado.

---

## Limitaciones y trabajo futuro

| Limitación actual | Mejora propuesta |
|-------------------|-----------------|
| Cinta de cobre se agrieta al doblar | Sustituir por hilo conductor cosido (e-textile) |
| Arduino externo — sistema no autónomo | Usar XIAO SAMD21 o ESP32-C3 integrado al textil |
| Una sola clase de respuesta visual (brillo/color) | Añadir patrones: pulso, chase, strobe según frecuencia dominante |
| Sin batería integrada | Batería LiPo plana de 100 mAh cosida al interior de la banda |
| Solo mide amplitud, no frecuencia | Añadir FFT con la librería `arduinoFFT` para reaccionar a graves vs agudos |

---

## Reflexión personal

La parte más reveladora de estos proyectos fue entender que construir un wearable funcional y construir un wearable *usable* son dos problemas distintos. En el laboratorio, con cables, fuente externa y el Arduino sobre la mesa, la demostración funciona. Sacado del laboratorio, el sistema no existe como objeto independiente.

Esa distancia entre "funciona en demos" y "funciona en uso real" es exactamente el problema que el proyecto Golf Sync intentó resolver con más rigor — y aún así quedó con el Arduino (ESP32-C3) conectado por cables jumper frágiles. Integrar electrónica en objetos blandos sigue siendo un problema abierto incluso en productos comerciales.

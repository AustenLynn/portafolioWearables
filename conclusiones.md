---
layout: default
title: Conclusiones
nav_order: 50
---

# Conclusiones

Reflexión crítica sobre el proceso de diseño y fabricación del wearable, y sobre el resultado obtenido.

---

## Reflexión sobre el proceso

El proceso de prototipado iterativo funcionó bien para este proyecto, porque permitió validar cada subsistema de forma independiente antes de la integración. Comenzar por el firmware y la lectura de sensores antes de abordar el BLE o el pipeline ML fue una decisión acertada: los problemas de dirección I2C y offset del giroscopio se detectaron en la etapa más barata de corregir.

Lo que cambiaría es la estrategia de captura de datos. Se invirtió demasiado tiempo en el pipeline de software antes de tener un dataset suficientemente grande. Con solo 5 swings "malos" en el dataset, el clasificador tiene un Bad recall de apenas 40%, lo que limita la utilidad práctica del sistema. Un ciclo de captura masiva al inicio (objetivo: 30 buenos + 30 malos antes de entrenar) habría producido un modelo más robusto.

---

## Reflexión sobre el resultado

El wearable logra los objetivos principales planteados en el [Concepto](concepto.md):

- ✅ **Captura continua a ~100 Hz** sin interrupciones durante el swing
- ✅ **Retroalimentación en < 3 s** desde el fin del golpe
- ✅ **Modelo personalizable** — el usuario puede relabelar swings y reentrenar desde la app
- ✅ **Funciona en cualquier red** gracias a la integración con Tailscale VPN
- ⚠️ **Clasificación aún limitada** — 76.2% de accuracy y 40% de recall en clase "malo" con el dataset actual

Los [Requisitos técnicos](metodologia-requisitos.md) de frecuencia de muestreo (≥ 70 Hz → logrado ~100 Hz), latencia (≤ 5 s → logrado ~2–3 s) y persistencia de datos (SQLite) se cumplen completamente.

---

## Aprendizajes clave

- **Aprendizaje 1 — BLE tiene restricciones no documentadas:** La limitación de 20 bytes por notificación en BLE NUS no está explícita en la documentación del ESP32 para Arduino. Hay que leer la especificación BLE o el código de la librería. El mecanismo de chunks fue el resultado de depurar datos corruptos en la app.
- **Aprendizaje 2 — El tamaño del dataset importa más que el modelo:** Cambiar de SVM a Random Forest mejoró el Bad recall de 0% a 40%, pero la ganancia real vendría de más datos. Con n=21, cualquier modelo tiene alta varianza.
- **Aprendizaje 3 — LOOCV es la única validación honesta con datasets pequeños:** K-fold con k=5 y n=21 produce pliegues de prueba con 4-5 muestras, demasiado ruidosos. LOOCV da la estimación de generalización más conservadora y honesta.
- **Aprendizaje 4 — Docker + Tailscale simplifica el deployment en campo:** El sistema funciona en una cancha de golf real sin configurar puertos en el router, gracias al túnel P2P de Tailscale entre la PC y el teléfono.

---

## Limitaciones y trabajo futuro

| Limitación actual | Mejora propuesta |
|-------------------|-----------------|
| Solo 5 swings "malos" en el dataset → Bad recall = 40% | Capturar 30+ swings malos usando la app; reentrenar con dataset balanceado |
| Clasificación binaria (BUENO/MALO) sin especificar el defecto | Etiquetar categorías de fallo: over-the-top, casting, over-swing, early extension |
| Segmentación basada en umbral fijo → fallos con diferentes portadores | Usar `scipy.signal.find_peaks` con `prominence` adaptativo por sesión |
| El modelo no mejora solo — requiere acción del usuario | Añadir sugerencia automática de reentrenamiento cuando se acumulan N swings nuevos etiquetados |
| PCB en protoboard → fragilidad mecánica | Diseñar PCB personalizada (KiCad) con componentes SMD para prototipo V2 |
| App solo en Android | Port a iOS con la misma base Flutter; Apple requiere permisos BLE distintos |

---

## Reflexión personal

Este proyecto fue la primera vez que integré hardware embebido, aprendizaje automático y desarrollo móvil en un único sistema funcional. La parte más gratificante fue ver el veredicto aparecer en el teléfono en tiempo real después de un golpe real — el ciclo completo de sensor → BLE → análisis → pantalla funcionando de extremo a extremo.

La parte más difícil fue aceptar que el modelo ML tiene limitaciones claras con tan pocos datos, y que la solución no es más ingeniería de features sino más datos de calidad. Esa lección sobre la primacía de los datos sobre los algoritmos es transferible a cualquier proyecto de ML embebido.

El proyecto también demostró que es posible construir un sistema de análisis deportivo funcional con hardware de < 30 USD y software open source, democratizando una tecnología que antes solo estaba disponible en instalaciones profesionales.

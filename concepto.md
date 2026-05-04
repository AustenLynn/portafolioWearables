---
layout: default
title: Concepto
nav_order: 10
---

# Concepto

> **Ponderación:** 10%

Esta sección describe la **intención de diseño** del proyecto wearable: la idea central, la motivación y los objetivos que guían todas las decisiones de diseño.

---

## Idea central

**Golf Swing Analyzer** es un wearable de análisis de golpe de golf. El dispositivo se coloca en el palo o en el cuerpo del golfista e integra un IMU (Unidad de Medición Inercial) de tres sensores — acelerómetro ADXL345, giroscopio ITG3200 y magnetómetro HMC5883L — que capturan el movimiento completo del swing a ~100 Hz.

Los datos crudos se transmiten en tiempo real por BLE (Bluetooth Low Energy, perfil Nordic UART Service) a una aplicación Android desarrollada en Flutter. La app detecta automáticamente el inicio y fin de cada swing, y envía el segmento capturado a un backend Python/FastAPI que extrae 19 características biomecánicas del golpe y lo clasifica como **BUENO** o **MALO** usando un modelo de Random Forest entrenado con datos del propio usuario. El resultado — veredicto, porcentaje de confianza y desglose de características — se muestra en el teléfono en tiempo real, permitiendo retroalimentación inmediata en el campo de práctica.

---

## Motivación

La técnica en el golf es difícil de mejorar sin un coach presente. Las cámaras de video requieren ángulos específicos y análisis manual posterior. Los sistemas comerciales de análisis de swing (ej. Trackman, Swing Catalyst) son extremadamente costosos y solo están disponibles en rangos de práctica especializados.

Este proyecto surge de la oportunidad de construir un sistema de análisis accesible, portable y personalizable, usando hardware de bajo costo (ESP32-C3, ~10 USD) y aprendizaje automático corriendo en una PC local. La naturaleza wearable del sensor elimina la necesidad de cámaras y permite captura en cualquier entorno.

---

## Objetivos de diseño

- **Objetivo 1 — Captura continua:** Medir yaw, pitch, roll y aceleración del club a ≥ 70 Hz de forma inalámbrica sin interrumpir el swing natural.
- **Objetivo 2 — Retroalimentación inmediata:** Clasificar el swing y mostrar el resultado al golfista en menos de 3 segundos después de terminar el golpe.
- **Objetivo 3 — Modelo personalizable:** Permitir que el usuario etiquete sus propios swings y reentrane el modelo desde la app, adaptando la clasificación a su técnica y cuerpo.
- **Objetivo 4 — Portabilidad:** El sistema completo (sensor + teléfono + backend en la nube o Tailscale) debe funcionar en cualquier cancha o driving range sin infraestructura especial.

---

## Usuario o contexto de uso

El dispositivo está diseñado para **golfistas aficionados y semi-profesionales** que desean mejorar su técnica de forma autónoma, sin depender de un instructor en cada sesión de práctica. El contexto de uso es el **driving range o campo de golf**:

- El golfista instala el sensor en la empuñadura del palo (grip) o en la muñeca.
- Conecta la app Android al sensor vía BLE.
- Golpea con normalidad: la app detecta el swing automáticamente.
- En segundos recibe el veredicto y el desglose de características (tempo, velocidad angular, aceleración en el impacto, rotación de hombros, etc.).

---

## Referencias e inspiración

El proyecto se inspira en sistemas académicos y comerciales de análisis de swing con IMU:

- **Ahmadi et al. (2009)** — "Automatic Activity Classification and Movement Assessment During a Sports Training Session Using Wearable Inertial Sensors" — primer trabajo relevante de clasificación automática de golpes con giroscopio.
- **Zepp Golf** (sensor comercial de muñeca) — demuestra que un IMU de bajo costo puede capturar características de swing relevantes.
- **GolfSense** y **Arccos Caddie** — muestran la viabilidad del perfil BLE para transmisión de datos de sensores en tiempo real hacia aplicaciones móviles.
- Filtro de fusión **Madgwick AHRS** — algoritmo eficiente de fusión de acelerómetro + giroscopio + magnetómetro empleado en el firmware.

---

## Siguiente sección

[Metodología de diseño](metodologia.md)

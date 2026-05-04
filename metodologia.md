---
layout: default
title: Metodología de diseño
nav_order: 20
has_children: true
---

# Metodología de diseño

Esta sección documenta el **proceso completo de diseño**: desde la definición del problema hasta la integración de ingeniería y estética en el wearable.

---

## Contenido de esta sección

| Subsección | Descripción |
|---|---|
| [Proceso de diseño](metodologia-proceso.md) | Metodología y etapas seguidas |
| [Requisitos técnicos](metodologia-requisitos.md) | Especificaciones funcionales y de rendimiento |
| [Arquitectura electrónica](metodologia-arquitectura.md) | Diagrama de bloques y selección de componentes |
| [Integración diseño–ingeniería](metodologia-integracion.md) | Cómo convergen la parte estética y la técnica |

---

## Resumen del proceso

Se adoptó una metodología de **Prototipado Iterativo** con cuatro subsistemas desarrollados en secuencia y validados de forma independiente antes de integrarse: (1) firmware ESP32 + sensores I2C, (2) transmisión BLE y captura de datos, (3) pipeline ML de segmentación + clasificación, y (4) API FastAPI + app Flutter. Cada iteración estuvo guiada por datos reales capturados con el propio sensor, permitiendo ajustar parámetros de segmentación, selección de features y arquitectura del clasificador en cada ciclo.

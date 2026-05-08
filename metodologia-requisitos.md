---
layout: default
title: Requisitos técnicos
parent: "Metodología de diseño"
nav_order: 2
---

# Requisitos técnicos

---

## Requisitos funcionales

| # | Requisito | Detalle | Prioridad |
|---|-----------|---------|-----------|
| RF-01 | Captura de movimiento | Leer aceleración (ax, ay, az), velocidad angular (gx, gy, gz) y orientación (yaw, pitch, roll) a ≥70 Hz de forma continua durante el swing. | Alta |
| RF-02 | Transmisión BLE | Enviar cada muestra IMU al teléfono en tiempo real como CSV por BLE NUS (Nordic UART Service), en chunks de 20 bytes. | Alta |
| RF-03 | Detección automática de swing | Segmentar el flujo de datos para identificar el inicio y el final de cada swing usando un umbral de velocidad angular (`omega_mag > 60 deg/s`). | Alta |
| RF-04 | Extracción de características | Calcular 19 features por swing detectado: tempo, velocidad angular, aceleración y ángulos de Euler. | Alta |
| RF-05 | Clasificación ML | Predecir GOOD (1) / BAD (0) con un modelo Random Forest entrenado sobre el dataset propio, devolviendo también la confianza de la predicción. | Alta |
| RF-06 | Retroalimentación en app | Mostrar veredicto, porcentaje de confianza, timeline de fases (backswing / downswing / follow-through) y desglose de features en menos de 2 s tras completar el swing. | Alta |
| RF-07 | Historial de swings | Almacenar cada swing clasificado en una base de datos SQLite persistente, consultable y filtrable por etiqueta (Good/Bad). | Media |
| RF-08 | Re-etiquetado y re-entrenamiento | Permitir al usuario corregir la etiqueta de cualquier swing histórico y lanzar el re-entrenamiento del modelo desde la app, sin necesidad de acceso a la PC. | Media |
| RF-09 | Operación fuera de red local | Funcionar desde cualquier red (campo de golf, datos móviles) sin abrir puertos de firewall, usando Tailscale como túnel cifrado. | Media |

---

## Requisitos no funcionales

| # | Requisito | Descripción | Valor objetivo |
|---|-----------|-------------|----------------|
| RNF-01 | Frecuencia de muestreo | El firmware debe mantener una tasa de muestreo estable. | ≥70 Hz (target 100 Hz) |
| RNF-02 | Latencia de retroalimentación | Tiempo entre el último sample del swing y el veredicto visible en pantalla. | ≤2 s |
| RNF-03 | Autonomía de batería | La batería LiPo debe sostener una sesión completa de práctica. | ≥1 h de transmisión BLE activa |
| RNF-04 | Rango BLE | El teléfono debe mantenerse conectado al sensor durante el swing. | ≥5 m |
| RNF-05 | Peso del módulo electrónico | El peso adicional no debe afectar el swing. | ≤30 g |
| RNF-06 | Tamaño del módulo | El stack debe caber en el dorso del guante sin sobresalir de los nudillos. | ≤60 mm × 40 mm × 15 mm |
| RNF-07 | Reproducibilidad | Swings idénticos capturados en condiciones iguales deben producir el mismo veredicto. | Consistencia ≥90% |

---

## Restricciones de diseño

- **Presupuesto:** módulos de prototipado disponibles en el laboratorio (ESP32-C3, ADXL345, ITG3200, HMC5883L, LiPo 3.7 V, cargador UNIT).
- **Plataforma objetivo:** Android (el teléfono disponible durante el desarrollo es Android; no se desarrolló versión iOS).
- **Dataset:** 21 swings (16 buenos, 5 malos) capturados en condiciones reales. Tamaño limitado por el tiempo disponible.
- **Tiempo de fabricación:** 6 semanas totales, incluyendo diseño, captura de datos, entrenamiento del modelo y construcción de los tres prototipos.
- **Backend:** debe correr en una PC existente (sin presupuesto para servidor cloud). Acceso remoto resuelto con Tailscale (ya instalado).

---

## Criterios de éxito

| Criterio | Cómo se mide | ¿Se cumplió? |
|----------|-------------|--------------|
| Pipeline end-to-end funcional | ESP32 → BLE → App → API → veredicto visible en pantalla | ✅ |
| Frecuencia de muestreo ≥70 Hz | Timestamps del CSV capturado (`t_ms` delta ≤14 ms) | ✅ (~100 Hz real) |
| Segmentación correcta de swings | Swings detectados sin fragmentaciones en las capturas del dataset | ✅ |
| Clasificador con accuracy >70% (LOOCV) | `train_classifier.py` → LOOCV accuracy | ✅ 76.2% Random Forest |
| App muestra desglose de features | Pantalla Result con phase timeline y feature breakdown | ✅ |
| Backend containerizado | `docker-compose up -d` levanta la API sin configuración manual | ✅ |
| Acceso remoto sin VPN de empresa | Tailscale conecta PC y teléfono en redes distintas | ✅ |

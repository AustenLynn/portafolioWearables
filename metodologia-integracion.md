---
layout: default
title: Integración diseño–ingeniería
parent: "Metodología de diseño"
nav_order: 4
---

# Integración diseño–ingeniería

Descripción de cómo la parte estética/funcional de diseño y la parte técnica de ingeniería convergen en el wearable.

---

## Decisiones de diseño con impacto técnico

| Decisión de diseño | Impacto técnico | Solución adoptada |
|--------------------|-----------------|-------------------|
| El sensor no debe alterar el agarre del palo | El módulo completo (ESP32 + 3 sensores) debe caber en un espacio ≤ 5×3 cm | Se usó una PCB de protoboard pequeña con los chips en posición horizontal; sin conectores voluminosos |
| El usuario no debe interactuar con el sensor durante el swing | El firmware no tiene botones; la app detecta el swing automáticamente | Detección por umbral en `omega_mag` implementada en Dart (app) y Python (backend) |
| El sistema debe funcionar al aire libre (campo de golf) | La red WiFi local puede no estar disponible | Se integró Tailscale VPN para conectar el teléfono (4G) con el backend en la PC del usuario |
| El resultado debe ser legible con un guante de golf y luz solar | La pantalla de resultado usa texto grande y colores alto contraste (verde/rojo) | UI en Flutter con banner de veredicto de altura ≥ 80 dp |

---

## Decisiones técnicas con impacto en el diseño

- **BLE con chunks de 20 bytes:** El protocolo NUS (Nordic UART Service) limita las notificaciones a 20 bytes. Esto requirió dividir cada línea CSV en fragmentos y rearmarlos en la app. El impacto en experiencia de usuario es nulo, pero añadió ~150 líneas de código en el servicio BLE de Flutter.
- **Calibración de giroscopio al inicio:** El ESP32 necesita ~2.5 segundos de reposo para calibrar offsets del giroscopio (500 muestras × 5 ms). Se aprovecha este tiempo para que el BLE inicie el advertising, así el usuario puede conectar la app mientras espera.
- **Madgwick en lugar de Mahony:** Madgwick produce menor drift en yaw a baja velocidad, crítico porque `yaw_at_top` (posición de backswing) es la feature con mayor separabilidad entre clases (Cohen's d = 1.67). Mahony sin magnetómetro derivaba varios grados por minuto.
- **SQLite local vs. base de datos remota:** Para garantizar que el sistema funcione sin internet, se eligió SQLite embebido en el contenedor Docker. Volumen montado para persistencia entre reinicios.

---

## Iteraciones de integración

### Iteración 1 — Offset de yaw al conectar BLE

**Problema:** El filtro Madgwick requiere que el sensor esté quieto al inicio para estabilizar la referencia. Si el golfista movía el palo mientras conectaba la app (durante el advertising BLE), el yaw inicial quedaba desreferenciado, corrompiendo `yaw_at_top`.

**Solución:** Se añadió el mensaje de calibración `# Calibrando gyro. No muevas el sensor...` transmitido por BLE antes de emitir el header CSV. La app muestra un diálogo bloqueante hasta recibir el header `t_ms,ax,ay,az,...`, garantizando que el sensor ya está estable.

### Iteración 2 — Falsa detección de swing en waggle

**Problema:** Los golfistas hacen "waggles" (movimientos de preparación) antes del golpe. El umbral `omega_mag > 60 deg/s` los detectaba como swings.

**Solución:** Se añadió `MIN_SWING_SAMPLES = 25` (~250 ms a 100 Hz). Los waggles son más cortos que un swing real y quedan filtrados automáticamente.

### Iteración 3 — Tiempo de respuesta del backend

**Problema:** En la primera versión, el backend recargaba el modelo desde disco en cada llamada a `POST /analyze`. Con el modelo de Random Forest (~200 árboles), esto añadía 400 ms por petición.

**Solución:** Se implementó `model_service.py` como un singleton que carga el modelo una vez al arrancar el contenedor y lo mantiene en memoria. La latencia de `/analyze` bajó de ~1200 ms a ~350 ms.

---

## Resultado de la integración

El sistema final integra cuatro componentes que se comunican de forma fluida:

1. **Sensor wearable** — pequeño, sin cables, fijado con velcro al grip del palo.
2. **App Android** — conecta automáticamente al sensor, muestra la gráfica de swing en vivo y recibe el veredicto en ≤ 3 s.
3. **Backend Docker** — corre en la PC del usuario, accesible desde cualquier red vía Tailscale.
4. **Pipeline ML** — modelo reentrenable desde la app, que mejora con cada sesión etiquetada.

La experiencia de uso es: _conectar la app → golpear con normalidad → ver el resultado_. No hay pasos adicionales para el golfista.

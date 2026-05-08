---
layout: default
title: Proceso de diseño
parent: "Metodología de diseño"
nav_order: 1
---

# Proceso de diseño

---

## Metodología empleada

Se adoptó un proceso de **prototipado iterativo orientado a datos** (*Data-Driven Iterative Prototyping*). Dado que el problema central era tanto de hardware (¿cómo montar el sensor en el guante?) como de software (¿qué datos son predictivos de un buen swing?), las iteraciones alternaron entre construcción física y validación de la pipeline de ML.

Cada prototipo físico generó datos reales que retroalimentaron el diseño electrónico siguiente; cada sesión de captura de datos reveló limitaciones en la segmentación y las features que modificaron el código de análisis. Diseño e ingeniería avanzaron en paralelo, no en secuencia.

---

## Etapas del proceso

### Etapa 1 — Investigación y empatía

Se realizó una encuesta a 57 personas sobre el uso de guantes de golf. Los hallazgos principales:
- Un guante dura entre **8 y 10 rondas** en uso moderado, y apenas **3 a 6 rondas** en uso intensivo.
- El **78% de los jugadores** simplemente descarta el guante cuando ya no sirve y compra uno nuevo.
- Solo el 22% lo sigue usando aunque haya perdido agarre.

Esto confirmó la viabilidad de integrar electrónica en el guante: el usuario ya asume que el guante es reemplazable. No hay resistencia psicológica a "modificarlo".

En paralelo, se revisó la literatura sobre detección de swing con IMU. El parámetro más discriminante encontrado fue el **tempo ratio** (duración backswing / duración downswing) — los buenos swings apuntan a una relación 3:1.

### Etapa 2 — Definición del problema

**Enunciado del problema:**

> El golfista aficionado no tiene acceso a retroalimentación técnica objetiva sobre la calidad de su swing durante la práctica. Los sistemas profesionales son costosos y fijos a una ubicación; las apps de video requieren un asistente. ¿Cómo dar retroalimentación en tiempo real, en el campo, sin depender de infraestructura externa?

Las restricciones de diseño resultantes:
- El sensor debe ir **en el cuerpo del jugador** (no en el palo) para no afectar el agarre.
- La retroalimentación debe llegar en **menos de 2 segundos** para que el jugador todavía "recuerda" el swing.
- El sistema debe funcionar **fuera de red WiFi** (campo de golf).

### Etapa 3 — Ideación

Se evaluaron tres conceptos de forma:

1. **Brazalete en muñeca** — sensor sobre la muñeca con correa foam. Rápido de construir. Desventaja: se mueve durante el swing, lecturas ruidosas.
2. **Guante con cinta** — sensor pegado al dorso del guante con cinta azul (masking tape). Primera aproximación al concepto final. Validó que la posición en el dorso de la mano captura el movimiento del swing correctamente.
3. **Guante con PCB integrado por costura** — sensor cosido directamente al textil del guante. Solución final adoptada en el prototipo v3.

### Etapa 4 — Prototipado (tres iteraciones)

**Prototipo v1 — Guante blanco con cinta azul (abril 20)**

Módulos electrónicos (IMU + cargador) pegados al dorso del guante FootJoy con masking tape azul. Objetivo: validar que la señal del sensor en esa posición permite detectar swings y diferenciar buenos de malos.

Resultado: señal usable. Los swings se detectaron correctamente. El sistema mecánico era frágil — la cinta se despega con el sudor — pero suficiente para capturar el dataset inicial.

![Prototipo v1 en mano](assets/img/evidencia/prototipo-v1-guante-blanco-frente.jpg)
*Figura — Prototipo v1: guante blanco con sensor IMU y cargador fijados con cinta azul.*

**Prototipo v2 — Stack completo autónomo (abril 22)**

Se añadió la batería LiPo directamente sobre el guante (sin cable USB). El stack completo (IMU + cargador UNIT + LiPo) fue fijado con cinta amarilla de mayor adherencia. Este prototipo fue el primero en funcionar sin conexión al PC.

![Prototipo v2 frontal](assets/img/evidencia/prototipo-v2-guante-blanco-frontal.jpg)
*Figura — Prototipo v2: stack electrónico completo integrado de forma autónoma.*

**Prototipo final — Guante verde con PCB cosido (abril 29)**

Se migró a un guante de tela verde sin dedos. El PCB fue integrado al dorso del guante mediante costura con hilo conductor. Este es el prototipo más compacto y estéticamente acabado.

![Prototipo final palma](assets/img/evidencia/prototipo-final-guante-verde-palma.jpg)
*Figura — Prototipo final: PCB personalizado cosido directamente al guante verde.*

### Etapa 5 — Evaluación e iteración

En cada prototipo se capturaron swings y se corrió la pipeline completa (segmentación → features → clasificador). Los hallazgos guiaron el diseño siguiente:

- **v1 → v2:** La cinta azul no resistía el sudor. Se cambió a cinta amarilla y se integró la batería para eliminar el cable de carga durante las sesiones.
- **v2 → final:** La cinta seguía siendo visible y poco profesional. Se adoptó la costura como método de fijación definitivo.
- **Pipeline ML:** Con 21 swings capturados (16 buenos, 5 malos) se entrenó un Random Forest con LOOCV. El recall en clase "bad" (40%) confirmó la necesidad de capturar más datos negativos.

---

## Cronograma

| Semana | Actividad principal |
|--------|---------------------|
| Semana 1 (Mar 9) | Primera captura de datos por BLE. Dataset inicial: 2 archivos de swings buenos. |
| Semana 2 (Mar 18) | Sesión intensiva: 8 archivos de swings buenos + 5 archivos de swings malos. Dataset completo para entrenamiento. |
| Semana 3 (Abr 20) | Prototipo v1 en guante blanco. Integración hardware + firmware + app. |
| Semana 4 (Abr 22) | Prototipo v2 con stack autónomo. Grabación del video de demostración. |
| Semana 5 (Abr 27–28) | Cierre del diseño gráfico (branding, investigación de usuario publicada). |
| Semana 6 (Abr 29) | Prototipo final en guante verde con PCB cosido. |

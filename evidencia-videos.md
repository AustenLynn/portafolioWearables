---
layout: default
title: Videos
parent: "Evidencia"
nav_order: 2
---

# Videos

Demostraciones en funcionamiento y registro en video del proceso de desarrollo del wearable Golf Swing Analyzer.

---

## Demostración de funcionamiento completo

_Video que muestra el ciclo completo: conectar la app al sensor vía BLE, realizar un swing real, observar la gráfica de `omega_mag` en tiempo real, y recibir el veredicto BUENO/MALO con el desglose de features en menos de 3 segundos._

**Duración aproximada:** 2–3 minutos

<!-- Opción A: Video almacenado en el repositorio (assets/videos/) -->
<video width="100%" controls>
  <source src="assets/videos/demo-funcionamiento.mp4" type="video/mp4">
  Tu navegador no soporta video HTML5.
</video>

---

## Demo de captura BLE y análisis por PC

_Video mostrando el script `capture_imu_ble.py` capturando swings por BLE desde PC, seguido de la ejecución del pipeline Python completo: `extract_features.py` → `train_classifier.py` → `classify_swing.py`._

**Duración aproximada:** 3–4 minutos

<video width="100%" controls>
  <source src="assets/videos/demo-pipeline-ml.mp4" type="video/mp4">
  Tu navegador no soporta video HTML5.
</video>

---

## Demo de reentrenamiento desde la app

_Video mostrando el flujo de mejora continua: relabelar un swing en la pantalla de Resultado → ir a Configuración → pulsar Reentrenar → el modelo actualizado clasifica el siguiente swing._

**Duración aproximada:** 1–2 minutos

<video width="100%" controls>
  <source src="assets/videos/demo-reentrenamiento.mp4" type="video/mp4">
  Tu navegador no soporta video HTML5.
</video>

---

## Video en YouTube (opcional)

<!-- Reemplaza VIDEO_ID con el ID real del video de YouTube -->
<!--
<iframe width="560" height="315"
  src="https://www.youtube.com/embed/VIDEO_ID"
  title="Golf Swing Analyzer — Demostración"
  frameborder="0"
  allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
  referrerpolicy="strict-origin-when-cross-origin"
  allowfullscreen>
</iframe>
-->

---

> **Nota:** Los videos se agregarán conforme se grabe la documentación en campo. Guarda los archivos `.mp4` en `assets/videos/` o publícalos en YouTube y usa el embed para reducir el tamaño del repositorio.

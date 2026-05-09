---
layout: default
title: Concepto
parent: "Pulseras"
nav_order: 2
---

# Concepto — Pulseras wearables

Esta sección cubre dos proyectos de pulseras independientes desarrollados como parte del mismo curso. Cada uno responde a una intención de diseño distinta y sirvió como exploración progresiva de la integración de electrónica en textiles.

---

## Pulsera 1 — Ejercicio de integración básica

### Intención de diseño

La primera pulsera no partió de un problema de usuario sino de un objetivo técnico: explorar cómo integrar electrónica discreta (LEDs individuales) en un sustrato textil o de foam sin que el resultado sea simplemente "una placa pegada a una tela".

La pregunta de diseño central fue: **¿cómo esconder los conductores y dar continuidad visual al wearable cuando el circuito es parte de la superficie?**

### Objetivo funcional

Encender 5 LEDs de colores en secuencia o simultáneamente, alimentados por una fuente externa, con los conductores integrados en el sustrato físico usando cinta de cobre como pista de señal.

### Usuario objetivo

No existe un usuario final definido. El proyecto fue un ejercicio de proceso: aprender a escalar desde un circuito en protoboard hacia un objeto que alguien podría llevar puesto.

---

## Pulsera 2 — Retroalimentación musical

### Intención de diseño

La segunda pulsera parte de un problema concreto: en contextos de música en vivo o práctica musical, el intérprete no tiene retroalimentación visual del volumen o dinámica de lo que está generando. La pulsera convierte el sonido ambiente en luz — el brillo y color de la tira LED varía en función de la amplitud captada por el micrófono.

La intención no es decorativa sino funcional: dar al usuario una lectura visual inmediata del nivel sonoro sin interrumpir lo que está haciendo con las manos.

### Objetivo funcional

Una tira NeoPixel (WS2812) integrada en una pulsera responde en tiempo real a la amplitud del sonido captado por un micrófono analógico conectado al Arduino. A mayor volumen, mayor brillo y desplazamiento de color hacia el rojo; en silencio, la tira se apaga o muestra un color tenue.

### Usuario objetivo

Músicos o estudiantes de música que quieren retroalimentación visual no intrusiva de su dinámica durante la práctica. También aplicable a ambientes de concierto o instalaciones interactivas de luz y sonido.

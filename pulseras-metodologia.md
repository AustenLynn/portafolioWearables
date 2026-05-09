---
layout: default
title: Metodología de diseño
parent: "Pulseras"
nav_order: 3
---

# Metodología de diseño — Pulseras

---

## Proceso de diseño

Ambas pulseras siguieron un proceso exploratorio lineal: una sola versión por proyecto, sin iteraciones formales. El énfasis estuvo en resolver el problema de integración — cómo llevar el circuito de la protoboard al objeto físico — más que en refinar la experiencia de usuario.

La secuencia de trabajo fue:

1. **Validar el circuito en protoboard** antes de tocar el sustrato físico
2. **Elegir el sustrato** (foam EVA, textil negro) en función del peso y flexibilidad necesarios
3. **Transferir el circuito al sustrato** con cinta de cobre como pista conductora
4. **Probar el conjunto integrado** conectado a fuente de laboratorio

No se realizaron pruebas con usuarios finales. La validación fue puramente técnica: ¿el circuito sigue funcionando cuando está integrado en el material?

---

## Arquitectura electrónica

### Pulsera 1 — 5 LEDs discretos

| Componente | Función |
|------------|---------|
| 5 LEDs de 5 mm (colores) | Salida visual |
| Resistencias (220 Ω c/u) | Limitación de corriente |
| Cinta de cobre conductora | Pistas de señal y tierra sobre foam |
| Foam EVA azul | Sustrato físico |
| Fuente externa (laboratorio) | Alimentación |

El circuito es puramente pasivo en cuanto a lógica: los 5 LEDs se conectan en paralelo con sus resistencias correspondientes entre la línea de alimentación y tierra. No hay microcontrolador ni control dinámico — encendido/apagado por conexión de la fuente.

```
VCC ──┬── R1 ── LED1
      ├── R2 ── LED2
      ├── R3 ── LED3
      ├── R4 ── LED4
      └── R5 ── LED5
                 │
                GND
(pistas sobre cinta de cobre en foam EVA)
```

### Pulsera 2 — NeoPixel reactiva a micrófono

| Componente | Función |
|------------|---------|
| Arduino MKR WiFi 1010 | Microcontrolador principal |
| Módulo micrófono analógico | Captura de amplitud de sonido |
| Tira NeoPixel WS2812 | Salida visual (LEDs RGB programables) |
| Batería / USB | Alimentación |

**Flujo de señal:**

```
Micrófono analógico
       │
       ▼ analogRead(A0)  [0 – 1023]
Arduino MKR WiFi 1010
       │
       ▼ map(valor, 0, 1023, 0, 255)
Tira NeoPixel WS2812
  → brillo y color según amplitud
```

El micrófono entrega una señal analógica proporcional al nivel de presión sonora. Arduino la muestrea con `analogRead()`, mapea el rango a 0–255 y lo usa para controlar el brillo de la tira NeoPixel. A mayor amplitud, mayor brillo; el color varía de azul (silencio) a rojo (volumen máximo) pasando por verde, creando un degradado visual intuitivo.

---

## Integración diseño–ingeniería

La principal tensión entre diseño e ingeniería en ambas pulseras fue la misma: **los conductores son rígidos, el sustrato es flexible**.

La cinta de cobre sobre foam funciona bien en posición plana, pero al doblar la pulsera alrededor de la muñeca la cinta tiende a despegarse o agrietarse en los puntos de curvatura máxima. Este problema no se resolvió completamente en ninguna de las dos versiones — es la limitación más importante del proceso.

Para la Pulsera 2, adicionalmente, el Arduino MKR WiFi es significativamente más grande que la pulsera de foam, lo que impidió integrar el microcontrolador al textil. El Arduino quedó externo (sobre la mesa de laboratorio) conectado al strip por cables, lo que significa que el prototipo final no es un wearable autónomo sino un demostrador funcional.

La integración real requeriría un microcontrolador mucho más pequeño (tipo XIAO o ESP32-C3 en formato bare chip) soldado directamente al sustrato.

---
layout: default
title: Proceso de manufactura
parent: "Pulseras"
nav_order: 4
---

# Proceso de manufactura — Pulseras

---

## Materiales utilizados

| Material | Pulsera 1 | Pulsera 2 |
|----------|-----------|-----------|
| Foam EVA azul | ✅ sustrato | — |
| Textil negro (neopreno/foam) | ✅ variante | ✅ sustrato |
| Cinta de cobre conductora | ✅ pistas | — |
| 5 LEDs discretos 5 mm | ✅ | — |
| Resistencias 220 Ω | ✅ | — |
| Tira NeoPixel WS2812 | — | ✅ |
| Arduino MKR WiFi 1010 | — | ✅ |
| Módulo micrófono analógico | — | ✅ |
| Cables jumper hembra-hembra | — | ✅ |
| Hilo conductor / costura | ✅ fijación | ✅ fijación |

**Herramientas:** tijeras de corte textil, calibrador digital, soldador de punta fina, multímetro, fuente de alimentación regulable de laboratorio.

---

## Pulsera 1 — Paso a paso

### Paso 1 — Diseño y corte del sustrato

Se cortó una tira de foam EVA azul con calibrador y tijeras a las dimensiones de una pulsera: ~25 cm de largo × 2.5 cm de ancho. El grosor del foam (~4 mm) da rigidez suficiente para sostener los LEDs sin que se doblen, pero aún permite flexión para ajustarse a la muñeca.

Se marcaron con lápiz las posiciones de los 5 LEDs, espaciados uniformemente a lo largo de la tira.

### Paso 2 — Pistas conductoras con cinta de cobre

Se adhirieron dos tiras de cinta de cobre a lo largo del foam: una para VCC y otra para GND. Las pistas corren paralelas al largo de la pulsera, con espacio suficiente para no cortocircuitarse.

En cada posición de LED se cortó la pista de GND y se dejó un pad de soldadura para la resistencia.

![Banda de foam con cinta de cobre y LEDs](assets/img/pulseras/fabricacion-banda-foam-cobre.jpg)
*Figura — Pulsera 1: banda de foam azul con pistas de cinta de cobre conductora y LEDs de 5 mm ensamblados.*

### Paso 3 — Integración de LEDs y resistencias

Se insertaron los 5 LEDs a través de pequeñas perforaciones en el foam, con el ánodo y cátodo doblados hacia atrás para hacer contacto con las pistas de cobre. Se soldó una resistencia de 220 Ω en serie con cada LED en la pista de GND.

Se verificó continuidad con multímetro antes de conectar a la fuente.

### Paso 4 — Prueba de alimentación

Se conectó la pulsera a la fuente de laboratorio a 3.3 V y se verificó que los 5 LEDs encendieran correctamente.

![Banda foam negra — prueba con fuente](assets/img/pulseras/fabricacion-banda-foam-negra-fuente.jpg)
*Figura — Variante de pulsera en foam negro conectada a fuente de alimentación regulable para prueba de consumo.*

---

## Pulsera 2 — Paso a paso

### Paso 1 — Montaje del circuito en protoboard

Se montó primero el circuito completo en protoboard para validarlo:

- Arduino MKR WiFi 1010 sobre mesa
- Módulo micrófono analógico conectado a pin A0 (señal) + 3.3 V + GND
- Tira NeoPixel WS2812 conectada a pin D6 (datos) + 5V + GND
- Se verificó que las lecturas del micrófono variaran con el sonido usando `Serial.println(analogRead(A0))`

![Arduino + breadboard + NeoPixel en protoboard](assets/img/pulseras/electronica-arduino-breadboard.jpg)
*Figura — Circuito de la Pulsera 2 validado en protoboard antes de integrar al sustrato.*

### Paso 2 — Programación del Arduino

Se cargó el sketch de retroalimentación musical:

```cpp
#include <Adafruit_NeoPixel.h>

#define MIC_PIN  A0
#define LED_PIN  6
#define NUM_LEDS 16

Adafruit_NeoPixel strip(NUM_LEDS, LED_PIN, NEO_GRB + NEO_KHZ800);

void setup() {
  strip.begin();
  strip.show();  // todos apagados al inicio
}

void loop() {
  int raw = analogRead(MIC_PIN);           // 0 – 1023
  int nivel = map(raw, 300, 900, 0, 255);  // ajustado al rango real del micrófono
  nivel = constrain(nivel, 0, 255);

  // Degradado azul (silencio) → verde → rojo (volumen alto)
  uint8_t r = nivel;
  uint8_t g = (nivel < 128) ? nivel * 2 : 255 - (nivel - 128) * 2;
  uint8_t b = 255 - nivel;

  strip.fill(strip.Color(r, g, b), 0, NUM_LEDS);
  strip.show();
  delay(10);
}
```

Los umbrales de `map()` (300–900 en lugar de 0–1023) se calibraron empíricamente midiendo el valor del micrófono en silencio (~300) y aplaudiendo fuerte junto al módulo (~900).

### Paso 3 — Integración de la tira NeoPixel al sustrato

Se adhirió la tira NeoPixel al interior de una banda de textil negro. La tira tiene adhesivo 3M en la parte posterior que facilita el pegado. Los cables de datos y alimentación se aseguraron con puntos de costura para evitar que se despegaran con el movimiento.

El Arduino MKR WiFi quedó externo al sustrato — conectado por cable — ya que su tamaño (61.5 × 25 mm) no permite integrarlo en la pulsera con los materiales disponibles.

![NeoPixel en muñeca con Arduino y app](assets/img/pulseras/electronica-neopixel-muñeca-arduino.jpg)
*Figura — Pulsera 2 en funcionamiento: tira NeoPixel encendida en muñeca durante prueba de retroalimentación musical.*

### Paso 4 — Validación funcional

Se realizó la prueba funcional en laboratorio:

| Condición | Comportamiento observado |
|-----------|--------------------------|
| Silencio | Tira apagada o color azul tenue |
| Conversación normal | Tira verde a media intensidad |
| Palmada fuerte | Tira roja a máxima intensidad |
| Música reproducida en bocina a 1 m | Variación visible sincronizada al ritmo |

La respuesta visual al sonido fue perceptible y sincronizada. La latencia (`delay(10)`) fue suficientemente baja para dar sensación de tiempo real.

---

## Desafíos y soluciones

| Desafío | Solución aplicada |
|---------|-------------------|
| Cinta de cobre se despega al doblar el foam | Se reforzaron los bordes con puntos de costura sobre la cinta |
| Micrófono saturado con ruido de fondo del laboratorio | Se ajustaron los umbrales de `map()` empíricamente |
| Arduino demasiado grande para integrar al textil | Se dejó externo; el sustrato solo contiene la tira LED |
| Corto en pistas de cobre al doblar la pulsera | Se aumentó la separación entre pistas VCC y GND a 3 mm |

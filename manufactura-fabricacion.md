---
layout: default
title: Procesos de fabricación
parent: "Proceso de manufactura"
nav_order: 1
---

# Procesos de fabricación

Descripción de los métodos y técnicas utilizados para construir la parte física del wearable.

---

## Materiales utilizados

| Material | Cantidad | Función |
|----------|----------|---------|
| ESP32-C3 (módulo) | 1 | Microcontrolador principal con BLE integrado |
| ADXL345 (breakout) | 1 | Acelerómetro 3-ejes |
| ITG3200 (breakout) | 1 | Giroscopio 3-ejes |
| HMC5883L (breakout) | 1 | Magnetómetro 3-ejes |
| Protoboard pequeña (3 cm × 7 cm) | 1 | Montaje temporal de componentes |
| Cable Dupont macho-hembra | ~10 | Conexiones I2C entre módulos |
| Batería LiPo 3.7 V 500 mAh | 1 | Alimentación portátil |
| Módulo de carga TP4056 con protección | 1 | Carga segura de la LiPo vía USB-C |
| Cinta de velcro doble cara | 30 cm | Fijación del sensor al grip del palo |
| Funda de tela elastizada | 1 | Protección del conjunto electrónico |

---

## Herramientas y maquinaria

- Soldador de punta fina (para pines de los breakouts)
- Multímetro digital (verificación de continuidad y voltajes)
- Computadora con Arduino IDE 2.x
- Cable USB-C para programación del ESP32-C3
- Tijeras y cinta doble cara

No se requirió corte láser ni impresión 3D en el prototipo V1. El alojamiento es una funda de tela elastizada cosida a mano.

---

## Paso a paso de fabricación

### Paso 1 — Preparación de los módulos breakout

Se soldaron tiras de pines macho a los módulos ADXL345, ITG3200 y HMC5883L para poder insertarlos en la protoboard. Los pines necesarios son: VCC, GND, SDA, SCL (y en el ADXL345, el pin SDO se conectó a VCC para fijar la dirección I2C en `0x53`).

### Paso 2 — Montaje en protoboard

Se posicionaron los tres módulos de sensores en la protoboard junto al ESP32-C3. Se realizaron las conexiones de bus I2C compartido:

- Todos los SDA → GPIO 6 del ESP32-C3
- Todos los SCL → GPIO 5 del ESP32-C3
- Todos los VCC → 3.3 V del ESP32-C3
- Todos los GND → GND común

Se verificó la ausencia de cortocircuitos con el multímetro antes de energizar.

### Paso 3 — Integración con la batería y módulo de carga

Se conectó la batería LiPo al módulo TP4056. La salida regulada del TP4056 (3.7–4.2 V) se conectó al pin de alimentación 5V/VIN del ESP32-C3 (que acepta 4.5–5.5 V; con la LiPo cargada a 4.2 V funciona correctamente).

### Paso 4 — Ensamblaje en funda

El conjunto protoboard + batería + módulo de carga se envolvió en una funda de tela elastizada de ~10 × 6 cm, cosida a mano con hilo elástico. La funda incluye una abertura lateral para el conector USB-C del TP4056 (recarga sin desmontar).

### Paso 5 — Fijación al palo de golf

Se pegó velcro de gancho en la parte trasera de la funda y velcro de bucle en la empuñadura del palo (debajo del grip de goma). El conjunto queda fijo durante el swing y se puede retirar en segundos.

---

## Desafíos y soluciones

| Desafío encontrado | Solución aplicada |
|--------------------|-------------------|
| La protoboard se movía ligeramente con el impacto del swing | Se agregó cinta de doble cara entre la protoboard y la funda; sin afectar las conexiones |
| El ESP32-C3 se reiniciaba al conectar la LiPo cargada (~4.2 V) | Se agregó un condensador de 100 µF en paralelo con la alimentación para absorber el pico de corriente inicial |
| Los cables Dupont se desconectaban con la vibración | Se aseguraron con una pequeña gota de silicona caliente en cada pin |

---

## Resultado físico

El prototipo final es un módulo compacto de aproximadamente **9 × 5 × 2 cm** con peso total de **~45 g** (incluyendo batería). Se fija al grip del palo con velcro y no interfiere perceptiblemente con el agarre normal.

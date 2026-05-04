---
layout: default
title: Implementación electrónica
parent: "Proceso de manufactura"
nav_order: 2
---

# Implementación electrónica

Documentación del ensamblaje del circuito, las pruebas de hardware y la integración de los componentes electrónicos en el wearable.

---

## Ensamblaje del circuito

Los tres módulos de sensores (ADXL345, ITG3200 y HMC5883L) comparten el bus I2C del ESP32-C3 mediante cableado paralelo. La dirección I2C de cada sensor es única, por lo que no hay conflictos en el bus:

| Sensor | Dirección I2C | Pin de configuración |
|--------|--------------|---------------------|
| ADXL345 | `0x53` | SDO → VCC (HIGH) |
| ITG3200 | `0x68` | AD0 → GND (LOW) |
| HMC5883L | `0x1E` | Fija (no configurable) |

El bus opera a 100 kHz (`Wire.setClock(100000)`) para compatibilidad con los tres sensores simultáneamente.

---

## Diagrama esquemático final

```
ESP32-C3
  GPIO5 (SCL) ─────┬──────── ADXL345 SCL
                   ├──────── ITG3200 SCL
                   └──────── HMC5883L SCL

  GPIO6 (SDA) ─────┬──────── ADXL345 SDA
                   ├──────── ITG3200 SDA
                   └──────── HMC5883L SDA

  3.3V ────────────┬──────── ADXL345 VCC
                   ├──────── ITG3200 VCC
                   └──────── HMC5883L VCC

  GND ─────────────┬──────── ADXL345 GND
                   ├──────── ITG3200 GND
                   └──────── HMC5883L GND

  ADXL345: SDO ──── VCC (dirección 0x53)
  ITG3200: AD0 ──── GND (dirección 0x68)

  VIN (5V-tolerante) ──── TP4056 OUT+ ──── LiPo 3.7V 500mAh
  GND ─────────────────── TP4056 OUT- ──── LiPo GND
```

---

## Pruebas de hardware

### Prueba 1 — Verificación de alimentación

| Parámetro | Valor esperado | Valor medido | ¿Pasa? |
|-----------|---------------|--------------|--------|
| Voltaje VCC (3.3 V rail del ESP32) | 3.3 V | 3.28 V | ✅ |
| Voltaje batería LiPo (cargada) | 4.2 V | 4.17 V | ✅ |
| Consumo en idle (BLE advertising) | ~80 mA | ~85 mA | ✅ |
| Consumo en captura activa (BLE conectado) | ~160 mA | ~168 mA | ✅ |

### Prueba 2 — Verificación de sensores (lectura I2C)

Se ejecutó el firmware con `Serial.println()` de las lecturas crudas antes de habilitar el filtro Madgwick. Se verificó:

- **ADXL345:** Con el sensor plano, az ≈ 1 g (9.8 m/s² ≈ 1 g con escala de 3.9 mg/LSB). ✅
- **ITG3200:** Con el sensor quieto, gx/gy/gz ≈ 0 °/s después de calibración. ✅ (Sin calibración: offset máximo medido ~12 °/s)
- **HMC5883L:** Valores de campo magnético cambian al rotar el sensor y se estabilizan en una dirección fija. ✅

### Prueba 3 — Verificación del filtro Madgwick

Con el sensor plano y quieto:
- pitch ≈ 0°, roll ≈ 0° ✅
- yaw varía lentamente (<0.5°/min de drift) ✅

Al rotar 90° sobre el eje Z (giro de yaw), el ángulo registrado cambió ~90° en la lectura de `filter.getYaw()`. ✅

### Prueba 4 — Verificación de transmisión BLE

Se conectó la app Python `capture_imu_ble.py` al ESP32 y se verificó que:
- Los chunks de 20 bytes se rearman correctamente en líneas CSV completas.
- La frecuencia de muestreo real (calculada de `t_ms`) es ~97–102 Hz. ✅
- No hay líneas corruptas ni pérdida de paquetes en condiciones de laboratorio (a <5 m). ✅

---

## Problemas encontrados y correcciones

| Problema | Causa | Corrección |
|----------|-------|------------|
| ESP32 no detectaba ADXL345 (dirección errónea) | SDO del ADXL345 flotante → dirección aleatoria | Conectar SDO a VCC explícitamente para fijar en `0x53` |
| Lecturas de HMC5883L daban 0x0000 | Registro 0x02 (modo) no inicializado | Agregar `writeRegister8(HMC5883L_ADDR, 0x02, 0x00)` en `initHMC5883L()` |
| Yaw derivaba varios grados/minuto | Filtro Madgwick solo con acelerómetro+giroscopio | Integrar las lecturas del HMC5883L en la llamada `filter.update(gx, gy, gz, ax, ay, az, mx, my, mz)` |
| Corte de alimentación al conectar la batería cargada | Pico de corriente al cargar capacitores del ESP32 | Agregar condensador de 100 µF / 6.3 V en el rail de alimentación |

---

## Resultado final del hardware

El hardware funciona de forma estable. La frecuencia de muestreo real es de ~100 Hz (rango medido: 97–103 Hz dependiendo de la carga del BLE). El drift de yaw con el filtro Madgwick y el magnetómetro es < 1° por minuto, suficiente para capturar una sesión completa de práctica sin recalibrar.

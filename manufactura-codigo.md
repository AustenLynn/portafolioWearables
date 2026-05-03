---
layout: default
title: Códigos de programación
parent: "Proceso de manufactura"
nav_order: 3
---

# Códigos de programación

Documentación del firmware, las librerías utilizadas y la lógica de control del wearable.

---

## Entorno de desarrollo

- **IDE / Editor:** …
- **Plataforma / Placa:** …
- **Lenguaje:** …

---

## Librerías utilizadas

| Librería | Versión | Función |
|----------|---------|---------|
| … | … | … |

---

## Arquitectura del software

_Describe la estructura del programa: módulos, flujo principal, interrupciones o eventos._

```
loop principal
├── leer sensores
├── procesar datos
└── actuar (LEDs, motores, comunicación)
```

---

## Código principal

_Pega aquí el código o los fragmentos más relevantes y explica qué hace cada sección._

```cpp
// Ejemplo de estructura básica (Arduino/C++)
void setup() {
  // Inicialización de pines y periféricos
}

void loop() {
  // Lógica principal del wearable
}
```

---

## Funciones clave

### `nombreFuncion()`

_Descripción de qué hace, parámetros y valor de retorno._

```cpp
// Insertar código
```

---

## Pruebas de software

| Prueba | Descripción | Resultado |
|--------|-------------|-----------|
| … | … | ✅ / ❌ |

---

## Repositorio del código

_Si el código está en un repositorio o carpeta separada, incluye el enlace o la ruta._

[Ver código fuente](assets/files/codigo-wearable.zip)

---
layout: default
title: Integración diseño–ingeniería
parent: "Metodología de diseño"
nav_order: 4
---

# Integración diseño–ingeniería

---

## Decisiones de diseño con impacto técnico

| Decisión de diseño | Impacto técnico | Solución adoptada |
|--------------------|-----------------|-------------------|
| El sensor debe ir en el **dorso del guante**, no en la muñeca ni en el palo | El sensor debe caber en ≈40×25 mm para no sobresalir de los nudillos, y el cableado no puede pasar por los dedos | Se escogió el módulo IMU 10 DOF como tarjeta compacta; la LiPo se ubica en el borde superior del guante donde hay más espacio |
| El guante debe ser **usable** durante el swing (no interferir con el agarre) | El módulo no puede tener conectores laterales que choquen con el palo; el cable USB no puede salir hacia la palma | El módulo de carga se orienta con el puerto USB-C hacia la muñeca, fuera del área de agarre |
| La **apariencia debe evolucionar** de prototipo a producto | Cinta → costura: el método de fijación determina el perfil del módulo y la rigidez frente al movimiento | v1: cinta azul (rápido, frágil) → v2: cinta amarilla (mayor adherencia) → final: costura con hilo conductor |
| El guante cambia con el uso — la electrónica debe ser **removible o reemplazable** | El PCB no puede estar soldado permanentemente al textil | En el prototipo final se usaron snap buttons (broches metálicos) para fijar el PCB al guante de forma desmontable |

---

## Decisiones técnicas con impacto en el diseño

**Tamaño del stack:** El ESP32-C3 con el módulo IMU apilado ocupa ≈40×25×15 mm. Esto forzó a extender el parche de sujeción hacia el dorso de la muñeca en lugar de concentrarlo sólo sobre los metacarpos. El resultado fue más visible pero más estable mecánicamente.

**BLE — una conexión a la vez:** El ESP32 soporta un único cliente BLE simultáneo. Esto significa que durante una sesión de captura con la app el PC está desconectado, y viceversa. Esta restricción técnica obliga a que el flujo de uso sea estrictamente secuencial: capturar → desconectar el PC → conectar el teléfono → analizar. El diseño de la app refleja esto: la pantalla Capture tiene un estado explícito "Conectando…" y muestra el nombre del dispositivo BLE encontrado antes de empezar.

**Filtro Madgwick — necesita estabilidad inicial:** El filtro AHRS tarda ~2 segundos en converger al inicio. El firmware lo resuelve con una **calibración de giroscopio automática** al encenderse (500 muestras estáticas). Esto se traduce en una restricción de uso: hay que encender el sensor sobre una superficie plana antes de ponerse el guante. El diseño de la carcasa debería incluir un LED de estado que indique "listo para usar" tras la calibración.

**Backend sin internet en el campo:** El backend corre en la PC del estudiante, no en la nube. Acceder a él desde el campo de golf requirió Tailscale. Esta restricción técnica condicionó el flujo de configuración de la app: la pantalla Settings tiene un campo de URL del backend y un botón "Test Connection" que el usuario debe configurar una sola vez al instalar la app.

---

## Iteraciones de integración

### Iteración 1 — Prototipo v1: módulo suelto con cinta azul

**Conflicto:** El sensor IMU y el cargador son dos módulos separados (no vienen integrados en un solo PCB). Hay que apilarlos y conectarlos con cables jumper cortos. La primera vez se intentó pegarlos al guante directamente con cinta de masking azul.

**Problema descubierto:** La cinta azul pierde adherencia con el sudor del guante. Después de 5-6 swings en campo, el módulo se desplazaba y la señal mostraba vibraciones no relacionadas con el swing.

**Resolución:** Para v2 se cambió a cinta amarilla de mayor adherencia (construcción). Como solución permanente, v3 adoptó la costura como método de fijación.

![Prototipo v1 en mano](assets/img/evidencia/prototipo-v1-guante-blanco-frente.jpg)
*Figura — Prototipo v1: sensor fijado con cinta azul. Funcional pero mecánicamente frágil.*

### Iteración 2 — Prototipo v2: batería integrada

**Conflicto:** En v1 la LiPo estaba separada del stack, conectada con cables colgantes. Durante el swing los cables se tensaban y podían desconectarse.

**Resolución:** En v2 la LiPo se pegó directamente sobre el guante al lado del stack electrónico, con la cinta amarilla cubriendo todo el conjunto. El sistema pasó a ser completamente autónomo (sin USB).

![Prototipo v2 con stack completo](assets/img/evidencia/prototipo-v2-guante-blanco-frontal.jpg)
*Figura — Prototipo v2: stack IMU + cargador + LiPo integrados sin cables externos.*

### Iteración 3 — Prototipo final: costura y guante verde

**Conflicto:** La cinta amarilla seguía siendo visible y poco profesional. Además, el guante FootJoy blanco es rígido — al cerrarse la mano el dorso se tensa y la cinta se despega en los bordes.

**Resolución:** Se adoptó un guante de tela verde sin dedos (más flexible, menor tensión en el dorso) y se sustituyó la cinta por costura directa con hilo de nylon. El perfil del módulo quedó reducido a la mitad de altura gracias a que el hilo comprime el PCB contra el textil.

![Prototipo final integrado](assets/img/evidencia/prototipo-final-guante-verde-palma.jpg)
*Figura — Prototipo final: PCB cosido directamente al guante verde. Módulo prácticamente al ras del textil.*

---

## Resultado de la integración

El prototipo final resuelve los tres conflictos principales identificados durante las iteraciones: el sensor no se mueve durante el swing (costura), la batería es autónoma y no necesita cables externos (stack integrado), y el perfil del módulo es suficientemente bajo para no interferir con el agarre del palo.

El sistema completo — firmware + backend + app — funcionó en pipeline end-to-end desde la primera sesión real de captura (marzo 2026), lo que permitió dedicar las semanas restantes a iterar sobre el diseño físico y mejorar el dataset de entrenamiento del clasificador.

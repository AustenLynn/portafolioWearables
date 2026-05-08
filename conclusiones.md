---
layout: default
title: Conclusiones
nav_order: 50
---

# Conclusiones

---

## Reflexión sobre el proceso

**Lo que funcionó bien:**

El enfoque de prototipado iterativo fue la decisión más acertada del proyecto. Empezar con un prototipo funcionalmente completo (aunque mecánicamente frágil) en la primera semana permitió capturar datos reales desde el inicio y validar la pipeline de ML con datos propios, no con datos de benchmark. El loop "construir → capturar → analizar → mejorar el diseño" fue productivo y evitó perder tiempo diseñando una carcasa perfecta antes de saber si el sensor producía señales útiles.

La arquitectura de software también fue una decisión correcta: reutilizar los mismos scripts Python como módulos en el backend FastAPI evitó código duplicado y garantizó que el pipeline que corrió en el notebook es exactamente el mismo que corre en producción.

**Lo que cambiaría:**

Se debería haber empezado a capturar swings "malos" desde la primera semana. El desbalanceo del dataset (16 buenos vs 5 malos) fue el factor limitante más importante del proyecto: el modelo Random Forest tiene recall del 40% en la clase "bad", lo que significa que falla en 3 de cada 5 swings defectuosos. Con un dataset balanceado de 30+30, el recall mejoraría sustancialmente.

También habría sido más eficiente construir un PCB integrado más temprano. Los cables jumper entre módulos fueron el punto de falla mecánico más frecuente durante las sesiones de captura.

---

## Reflexión sobre el resultado

**¿El wearable logra los objetivos del [Concepto](concepto.md)?**

| Objetivo | ¿Cumplido? | Observación |
|----------|-----------|-------------|
| Capturar movimiento a ≥70 Hz | ✅ | ~100 Hz real |
| Transmitir vía BLE sin cables | ✅ | BLE NUS estable hasta 5 m |
| Clasificar GOOD/BAD en tiempo real | ✅ | Veredicto en ≤2 s |
| Mostrar desglose de features en la app | ✅ | Phase timeline + feature breakdown |
| Mantener historial de mejora | ✅ | SQLite persistente + pantalla History |
| Re-entrenar el modelo desde la app | ✅ | Botón Retrain en Settings |
| Operar fuera de red local | ✅ | Tailscale resuelve el acceso remoto |

El sistema cumple todos los requisitos funcionales planteados. La limitación es estadística, no técnica: el clasificador necesita más datos, especialmente de la clase "bad".

---

## Aprendizajes clave

- **El tempo ratio (backswing/downswing) no es el mejor predictor de un buen swing en este dataset.** Las features de aceleración (`acc_mean`, `acc_std`) y velocidad angular media (`omega_mean`) resultaron más discriminantes. Los malos swings son aproximadamente el doble de rápidos en promedio que los buenos — el jugador "golpea fuerte" en lugar de "golpear bien".

- **LOOCV es el único validador honesto con 21 muestras.** Con este tamaño de dataset, cualquier split train/test clásico puede producir resultados artificialmente buenos o malos. La validación LOOCV (sacar una muestra, entrenar en las demás, predecir la sacada) es la única estimación de rendimiento estadísticamente válida.

- **BLE + Flutter es una combinación madura y confiable.** `flutter_blue_plus` maneja correctamente la reconexión, el MTU negotiation y los chunks de 20 bytes. No hubo problemas de compatibilidad entre el ESP32 y Android en ningún dispositivo probado.

- **Docker simplifica el despliegue en equipo.** Con `docker-compose up -d` cualquier persona puede levantar el backend en su PC sin instalar dependencias Python manualmente. El volumen persistente garantiza que el historial de swings y el modelo entrenado no se pierden al reiniciar el contenedor.

- **El guante como interfaz wearable es viable.** La investigación de usuario lo confirmó desde el principio: el 78% de los jugadores ya trata el guante como desechable. Integrar electrónica en él no genera resistencia psicológica. El reto es mecánico (fijación estable) y de tamaño, no de aceptación del usuario.

---

## Limitaciones y trabajo futuro

| Limitación actual | Mejora propuesta |
|-------------------|-----------------|
| Solo 5 swings "malos" → recall del clasificador 40% | Capturar 25+ malos adicionales. Target: 30 buenos + 30 malos |
| Todos los swings malos son una sola clase | Etiquetar por tipo de falla: over-the-top, casting, over-swing, early extension |
| El segmentador puede partir un swing largo en dos | Usar `scipy.signal.find_peaks` con `prominence` y `distance` mínima en lugar de hysteresis simple |
| PCB construido con cables jumper (frágil) | Diseñar PCB integrado con ESP32-C3 + ADXL345 + ITG3200 + HMC5883L + cargador en una sola tarjeta |
| La app solo funciona en Android | Compilar la misma base de código Flutter para iOS |
| El modelo LOOCV con 21 muestras no es estadísticamente robusto | Con 60+ muestras, cambiar a stratified k-fold y hacer grid search de hiperparámetros |
| El backend requiere PC encendida | Desplegar el backend en Raspberry Pi o servidor cloud de bajo costo |

---

## Reflexión personal

Este proyecto demostró que es posible construir un sistema de análisis de movimiento deportivo completamente funcional — desde el sensor hasta la app — con hardware de laboratorio estándar, código abierto y en pocas semanas.

El límite más frustrante fue la escasez de datos negativos. La intuición diría que un swing "malo" es fácil de producir intencionalmente, pero en la práctica resulta que los golpes deliberadamente malos no se parecen a los malos de un jugador real que intenta mejorar. Esa distinción obliga a capturar datos en contexto real, con un jugador real intentando hacer bien el swing, y que le salga mal — lo cual requiere tiempo y condiciones de campo.

La parte más satisfactoria fue ver el pipeline funcionar end-to-end por primera vez: hacer un swing, ver el veredicto GOOD en la pantalla del teléfono en menos de dos segundos, y entender qué feature hizo la diferencia. Eso es exactamente el tipo de retroalimentación que el golfista aficionado necesita y que hoy no tiene acceso.

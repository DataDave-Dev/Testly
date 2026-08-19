# Idea y decisiones — Testly

Fecha: 2026-08-18
Estado: mapeo cerrado, listo para planear implementación

## Pitch

**Testly.** Plataforma web educativa. El estudiante pega una función o fragmento de código.
La plataforma genera **pruebas unitarias** para ese código y le explica **por qué
se prueba así**: qué caso cubre cada prueba, qué podría romperse ahí y cómo se
eligieron los casos.

No es un generador de pruebas a secas. El objetivo es que después de usarla
varias veces el estudiante sepa escribir sus propias pruebas sin ella. Por eso
la explicación no es un adorno alrededor del código generado: es el producto.

Un generador que solo escupe un archivo de pruebas enseña a depender de él. La
diferencia está en que el estudiante entienda el criterio con el que se eligieron
los casos, no solo el resultado.

## Flujo base

1. El estudiante llega a la landing: qué hace la plataforma, un poco de teoría
   de pruebas unitarias, ejemplos
2. Pasa a la pantalla principal **sin necesidad de cuenta**
3. Pega su código y elige (o se detecta) el lenguaje
4. El backend envía el código al modelo con un prompt pedagógico
5. Se devuelve el archivo de pruebas más la explicación de cada caso
6. El estudiante puede copiar o descargar el archivo
7. Si quiere guardarlo, crea una sesión (magic link, sin contraseña) y a partir
   de ahí sus generaciones quedan en su historial

## Decisiones tomadas

| Tema | Decisión | Razón |
|---|---|---|
| Producto | Pruebas unitarias generadas + explicación del criterio de cada caso | El estudiante aprende a probar, no solo recibe pruebas |
| Motor | LLM vía API | Cubre los 4 lenguajes y sus frameworks sin escribir un generador por lenguaje |
| Ejecución de código | **No se ejecuta**, ni el código ni las pruebas generadas | Elimina el mayor riesgo de seguridad y toda la infraestructura de sandbox |
| Verificación de las pruebas | Reglas por framework (RF-20) + heurística de pruebas tautológicas (RF-24), no ejecución | Consecuencia directa de no ejecutar. Ver riesgos 1 y 4 |
| Cuentas | **Opcional.** Sin cuenta: 3 generaciones/día por IP, sin historial. Con cuenta: cuota mayor + historial | Bajar la fricción para que la landing convierta; el costo se controla por IP + Turnstile en vez de por login |
| Entrega | Copiar y descargar el archivo de pruebas | Sin esto el estudiante retranscribe a mano y la herramienta estorba |
| Stack | Astro (SSR) + React + Tailwind + Turso + Better Auth, en Cloudflare Workers | Preferencia del equipo; comparación por capa en [02-arquitectura.md](02-arquitectura.md). Cloudflare sobre Vercel: sin límite de tiempo de pared en el streaming. Turso porque es SQLite en el edge, mismo criterio de latencia |
| Lenguajes | Python, JavaScript/TypeScript (MVP). Java y C/C++ quedan fuera del alcance inicial | Acota el conjunto de evaluación y la calibración de la Fase 2 a lo que el equipo puede validar bien en un semestre; Java y C/C++ quedan como trabajo futuro |
| Frameworks | pytest, Vitest (MVP). JUnit 5 y GoogleTest quedan fuera del alcance inicial junto con Java y C/C++ | Uno por defecto por lenguaje, cambiable por el usuario |
| Contexto | Proyecto escolar / FIME | Define el formato de la documentación y el alcance realista |

Detección de errores y observaciones de estilo quedan fuera. El modelo puede
mencionar que algo es difícil de probar cuando viene al caso —es información
sobre el diseño del código—, pero no se promete como sección.

## Riesgos identificados desde el inicio

1. **Pruebas que no compilan o no pasan.** Como no se ejecuta nada, nadie
   garantiza que el archivo generado corra. Un import mal puesto o un nombre de
   función equivocado y el estudiante recibe algo roto. Es el riesgo central del
   producto y la interfaz tiene que ser honesta al respecto.
2. **Costo por uso.** Cada análisis es una llamada de pago, y generar código es
   salida larga: más cara que explicar. Ver [03-costos.md](03-costos.md).
3. **Uso como "hazme la tarea".** Más agudo que en la versión anterior de esta
   idea: si la tarea del profesor es *escribir pruebas unitarias*, esta
   herramienta la hace. Se aborda de frente en el PRD, sección 9, sin fingir que
   se resuelve.
4. **Pruebas de mentira.** El modelo puede generar pruebas que pasan siempre
   (`assert True`, aserciones sobre nada, mocks que devuelven justo lo que se
   afirma). Se ven bien y no prueban nada. El conjunto de evaluación tiene que
   buscar esto explícitamente. **Mitigación (RF-24):** heurística estática que
   detecta estos patrones y marca la prueba como sospechosa antes de
   entregarla, sin ejecutar nada — mismo criterio de costo que RF-20.
5. **Abuso del flujo sin cuenta.** Al no exigir login para generar, alguien
   puede intentar automatizar peticiones contra la cuota gratis. Se mitiga con
   Cloudflare (regla de rate limiting por IP + Turnstile en el botón de
   generar), no con identidad. Ver [02-arquitectura.md](02-arquitectura.md),
   sección 7.

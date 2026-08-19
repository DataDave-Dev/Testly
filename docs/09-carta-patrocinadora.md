# Carta patrocinadora (Project Charter)

Fecha: 2026-08-19
Versión: 0.2

Adaptada de la carta sponsor original al estado actual del proyecto
documentado en `docs/`. Fuente por sección anotada al margen.

## Título del proyecto

**Testly — Generador pedagógico de pruebas unitarias (IA Beta)**

Fuente: [00-idea.md](00-idea.md), [01-prd.md](01-prd.md).

## Identificación del beneficio del proyecto

- **Acompañamiento y razonamiento pedagógico:** a diferencia de un generador
  comercial, ayuda al estudiante a comprender el criterio con el que se
  eligen los casos de prueba, no solo a recibir un archivo de pruebas.
- **Cobertura integral de casos:** identifica y explica caso feliz, valores
  límite, entradas inválidas y manejo de errores, evitando que el estudiante
  pruebe solo el camino feliz.
- **Honestidad frente a pruebas de mentira:** a diferencia de pegar el código
  directo a un asistente genérico, el sistema aplica una heurística (RF-24)
  que detecta y marca pruebas tautológicas (`assert True` y similares) antes
  de entregarlas, sin necesidad de ejecutar código.

Fuente: [00-idea.md](00-idea.md) sección "Pitch", [01-prd.md](01-prd.md)
sección 1-2.

## Lista de documentos o productos esperados

1. **Plataforma web funcional (Astro + React, Cloudflare Workers):** landing,
   editor para pegar código y despliegue del resultado en streaming.
2. **Integración con la API del modelo (Sonnet 5):** `lib/llm/`, prompt de
   sistema y generación del archivo de pruebas.
3. **Explicación por caso de prueba:** parte del mismo flujo de generación,
   no un módulo separado — qué cubre cada caso y por qué se eligió.
4. **Cuentas opcionales y persistencia:** Better Auth (magic link) + Turso,
   historial de generaciones.
5. **Repositorio de código:** control de versiones y gestión del proyecto.
6. **Documentación del proyecto:** esta carpeta `docs/` (PRD, arquitectura,
   costos, plan, metodología, límites).
7. **Presentación final del prototipo.**

Fuente: [02-arquitectura.md](02-arquitectura.md), [01-prd.md](01-prd.md)
sección 5.

## Identificación de medidas clave de éxito

1. **Generación correcta:** el endpoint `/api/generar` procesa el código y
   devuelve pruebas ejecutables en el framework elegido.
2. **Cobertura de casos:** sobre las 20 funciones del conjunto de evaluación,
   las pruebas cubren caso feliz, borde y error en al menos 15.
3. **Validez:** al menos 17 de esas 20 corren sin errores de sintaxis, import
   o nombre.
4. **Cero pruebas tautológicas** en la muestra evaluada.
5. **Usabilidad sin fricción:** el estudiante genera y descarga sin gestionar
   la API directamente.
6. **Cumplimiento iterativo en Scrum:** entregas revisadas al cierre de cada
   sprint, dentro de la fecha límite.
7. **Uso real:** al menos 10 estudiantes ajenos al equipo generan al menos
   una vez.

Fuente: [01-prd.md](01-prd.md) sección 9.

## Prioridad en relación a otros proyectos

Prioridad alta. Proyecto central del periodo académico actual, con fecha
límite fija (ver abajo). Su ejecución tiene precedencia sobre otros
compromisos del equipo dentro del semestre.

## Fechas de inicio, fin y momentos clave

**Fecha límite: 2026-12-01**, sin excepción. Desde el inicio (2026-08-18) son
~15 semanas, ~7 sprints de 2 semanas para las 6 fases siguientes. El reparto
exacto de sprints por fase se confirma en Sprint Planning, no está fijado de
antemano (ver [07-metodologia.md](07-metodologia.md) sección 3).

| Fase | Contenido | Estimado |
|---|---|---|
| Fase 0 — Preparación | Equipo, presupuesto, conjunto de evaluación | 1 sprint |
| Fase 1 — Núcleo de la generación | Endpoint `/api/generar`, streaming, protección desde el día uno | 2 sprints |
| Fase 2 — Calibración | Medir `effort` sobre Sonnet 5, validar contra el conjunto de evaluación | 1 sprint |
| Fase 3 — Cuentas opcionales y persistencia | Better Auth, Turso, historial | 1 sprint |
| Fase 4 — Interfaz | Tokens de diseño, landing, pantallas | 1 sprint |
| Fase 5 — Prueba con usuarios y cierre | Prueba con 10+ estudiantes, documentación final | 1 sprint |

Si el ritmo no alcanza, se recorta la Fase 4 antes que la Fase 2 o la Fase 5.

Fuente: [04-plan.md](04-plan.md), [08-limites.md](08-limites.md) sección 1.

## Administrador

**Product Owner asignado; Scrum Master pendiente.**

| Rol | Responsabilidad | Quién |
|---|---|---|
| Product Owner | Prioriza el backlog contra el PRD, decide qué entra a cada sprint | Alonso David de León Rodarte |
| Scrum Master | Facilita ceremonias, quita bloqueos | Por asignar |
| Equipo de desarrollo | Implementa el sprint backlog | Todos, incluido el PO |

En equipo de 6, PO y Scrum Master también pueden tomar tareas de desarrollo.
Alonso es PO y developer a la vez: no autoaprueba su propio código, otra
persona del equipo revisa igual.

Fuente: [07-metodologia.md](07-metodologia.md) sección 2.

## Miembros

| Nombre | Matrícula |
|---|---|
| Dante Mauricio Calderón Barboza | 2021203 |
| Alonso David de León Rodarte | 2173878 |
| Perla Rubí Leal Leal | 2039055 |
| Ángel Alberto Isaac Meza | 2064850 |
| Osmar Emilio Solís Olivares | 2044259 |
| Isela Esmeralda Guerrero Martínez | 2018532 |

Fuente: [07-metodologia.md](07-metodologia.md) sección 2.

## Firma

Firman los dos roles de liderazgo Scrum. No hay sponsor externo en este
proyecto (escolar, sin empresa/cliente), por lo que la firma representa el
compromiso del equipo con lo establecido en esta carta.

| Rol | Nombre | Firma | Fecha |
|---|---|---|---|
| Product Owner | Alonso David de León Rodarte | | |
| Scrum Master | Por asignar | | |

## Historial de versiones

| Versión | Fecha | Cambio |
|---|---|---|
| 0.1 | 2026-08-19 | Primera versión, adaptada de la carta sponsor original al estado actual del proyecto. Roles Scrum sin asignar persona, dejado pendiente a propósito |
| 0.2 | 2026-08-19 | Se agrega beneficio "Honestidad frente a pruebas de mentira" (RF-24), heurística estática que detecta pruebas tautológicas sin ejecutar código |
| 0.3 | 2026-08-19 | **Product Owner asignado: Alonso David de León Rodarte**, también developer. Scrum Master sigue por asignar |
| 0.4 | 2026-08-19 | Se agrega sección Firma (PO + Scrum Master), elemento que faltaba según la presentación de apoyo de la carta sponsor ([docs/apoyo](apoyo/)) |

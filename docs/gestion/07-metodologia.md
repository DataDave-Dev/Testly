# Metodología de desarrollo — Testly

Fecha: 2026-08-18
Versión: 0.6

## 1. Metodología elegida: Scrum

**Scrum**, decisión del equipo. Se monta sobre las fases ya definidas en
[04-plan.md](04-plan.md): cada fase se reparte en uno o más sprints; el
entregable de la fase coincide con el Sprint Review del sprint que la cierra.

## 2. Equipo

| Nombre | Matrícula |
|---|---|
| Dante Mauricio Calderón Barboza | 2021203 |
| Alonso David de León Rodarte | 2173878 |
| Perla Rubí Leal Leal | 2039055 |
| Ángel Alberto Isaac Meza | 2064850 |
| Osmar Emilio Solís Olivares | 2044259 |
| Isela Esmeralda Guerrero Martínez | 2018532 |

### Roles Scrum

Propuesta, el equipo confirma quién toma cada uno:

| Rol | Responsabilidad | Quién |
|---|---|---|
| Product Owner | Prioriza el backlog contra el PRD ([01-prd.md](../producto/01-prd.md)), decide qué entra a cada sprint, es la voz de "qué necesita el proyecto" frente al profesor | Osmar Emilio Solís Olivares |
| Scrum Master | Facilita las ceremonias, quita bloqueos, cuida que el sprint no se sobrecargue | Isela Esmeralda Guerrero Martínez |
| Equipo de desarrollo | El resto: implementa el sprint backlog | Todos, incluido el PO |

En equipo de 6, PO y Scrum Master pueden también tomar tareas de desarrollo;
no son roles de tiempo completo aquí. Osmar es PO y developer a la vez: no
autoaprueba su propio código, otra persona del equipo revisa igual.

### Áreas técnicas (de [02-arquitectura.md](../arquitectura/02-arquitectura.md))

Reparto sugerido por pieza del stack, para saber quién arranca qué en el
primer sprint de cada fase — no es asignación fija de todo el semestre:

| Área | Cubre |
|---|---|
| Frontend / islas | Páginas Astro, islas React (`CodeInput`, `ResultStream`), Tailwind y tokens (Fase 4) |
| LLM y prompt | `lib/llm/`, diseño y calibración del prompt, conjunto de evaluación (Fase 0 y 2) |
| Auth y base de datos | Turso, Better Auth, `lib/db/generations.ts`, `lib/db/anonQuota.ts` (Fase 3) |
| Backend y validación | Endpoint `/api/generar`, `frameworks.ts`, `extraerTest.ts`, aserciones por framework (Fase 1) |
| Seguridad y acceso | Middleware, Turnstile, rate limiting de Cloudflare, cuotas (Fase 1 y 3) |
| Documentación y QA | Mantener `docs/`, coordinar la prueba con usuarios de la Fase 5, correr a mano el conjunto de evaluación en Fase 2 |

## 3. Sprints

- **Duración: 2 semanas.** Ajustable si el calendario escolar lo exige (ver
  nota abajo).
- Cada fase de 04-plan.md se divide en 1 o 2 sprints según su carga. Por
  ejemplo, la Fase 1 (núcleo de la generación) probablemente necesita 2
  sprints; la Fase 0 (preparación) puede caber en uno.
- Un sprint no cruza el límite de una fase: no se mezclan tareas de la Fase 1
  con tareas de la Fase 3 en el mismo sprint, porque la Fase 3 depende de que
  la Fase 1 esté cerrada.

**Fecha límite: 2026-12-01** (ver [04-plan.md](04-plan.md)). Desde hoy
(2026-08-18) son ~15 semanas, es decir **~7 sprints de 2 semanas** para las 6
fases. Ajustado — la Fase 1 sola se estimó en 2 sprints — así que no hay
margen para sprints que no cierren nada. Si el ritmo real no alcanza, se
recorta la Fase 4 antes que la 2 o la 5 (criterio en 04-plan.md).

**Capacidad confirmada:** al menos 1-2 personas pueden dedicar ~12 horas/semana
sin problema. Eso da un piso de **24 horas/semana de equipo** (48 horas por
sprint de 2 semanas) contando solo a esas 1-2 personas, sobre las ~15 semanas
disponibles hasta el 2026-12-01: ~360 horas-persona de piso.

**Sigue pendiente:** quiénes son esas 1-2 personas, y cuántas horas/semana
pueden dar las otras 4-5. Sin eso, el piso de 24h/semana es la única cifra
confiable para el Sprint Planning — todo lo demás es capacidad no
garantizada.

## 4. Ceremonias

| Ceremonia | Cuándo | Qué |
|---|---|---|
| Sprint Planning | Primer día del sprint | Se toman tareas del Product Backlog al Sprint Backlog, acotadas a lo que cabe en 2 semanas |
| Daily standup | Async, por chat del equipo (no llamada obligatoria) | Qué hice, qué haré, qué me bloquea. Reemplaza junta diaria por horarios de estudiante |
| Sprint Review | Último día del sprint | Se muestra lo terminado, funcionando. Coincide con el entregable de la fase cuando el sprint la cierra |
| Sprint Retrospective | Después del Review | Qué funcionó, qué no, qué se ajusta para el siguiente sprint |

## 5. Backlog

- **Product Backlog:** sale de los requisitos funcionales y no funcionales de
  [01-prd.md](../producto/01-prd.md) (RF-01 a RF-23, RNF-01 a RNF-08) más las tareas
  técnicas de cada fase en 04-plan.md. El Product Owner lo prioriza.
- **Sprint Backlog:** subconjunto tomado en Sprint Planning, acotado a la fase
  vigente.
- Cada tarea del backlog referencia el RF/RNF o la sección de
  [02-arquitectura.md](../arquitectura/02-arquitectura.md) que resuelve.

## 6. Definición de terminado (Definition of Done)

Una tarea se marca hecha cuando:

- El código está revisado por al menos otra persona del equipo
- Tiene pruebas unitarias si toca `src/lib/` (ver sección 2.12 de
  [02-arquitectura.md](../arquitectura/02-arquitectura.md))
- Si cambia una decisión ya documentada, el doc correspondiente en `docs/` se
  actualiza en el mismo sprint, no después

## 7. Herramienta

**GitHub Projects**, con cada sprint como iteración. Evita una herramienta más
que sincronizar aparte del repo de código.

## Historial de versiones

| Versión | Fecha | Cambio |
|---|---|---|
| 0.1 | 2026-08-18 | Primera versión: propuesta de Kanban, sin confirmar por el equipo |
| 0.2 | 2026-08-18 | **Cambio de metodología: Scrum**, decisión del equipo. Sprints de 2 semanas sobre las fases de 04-plan.md, roles Scrum, ceremonias, backlog y Definition of Done |
| 0.3 | 2026-08-18 | Fecha límite confirmada (2026-12-01): ~7 sprints disponibles para las 6 fases, calendario ajustado. Horas/semana por persona queda pendiente |
| 0.4 | 2026-08-18 | Capacidad parcial confirmada: 1-2 personas a ~12h/semana (piso de 24h/semana de equipo). Falta identificar a esas personas y la disponibilidad de las 4-5 restantes |
| 0.5 | 2026-08-19 | **Product Owner asignado: Alonso David de León Rodarte**, también developer. Scrum Master sigue por asignar |
| 0.6 | 2026-08-19 | **Scrum Master asignada: Isela Esmeralda Guerrero Martínez** |
| 0.7 | 2026-08-19 | **Product Owner asignado: Osmar Emilio Solís Olivares** |

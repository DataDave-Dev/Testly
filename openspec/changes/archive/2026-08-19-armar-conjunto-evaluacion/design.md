## Context

Testly delega la generación de pruebas a un modelo de lenguaje (Sonnet 5). La
única forma de saber si eso funciona — y de calibrar `effort` sin quemar el
presupuesto de $20 USD ([03-costos.md](../../../docs/costos/03-costos.md)) —
es correr un conjunto fijo de funciones contra el modelo y comparar lo
generado contra una lista de casos esperados escrita a mano de antemano. Ese
conjunto es lo que esta propuesta define.

No existe código de producto todavía (Fase 1 no ha arrancado, no hay `src/`
en el repo). Este es un entregable de datos/documentación puro de la Fase 0,
consumido después por un script de automatización en Fase 2
([04-plan.md](../../../docs/gestion/04-plan.md), sección Fase 2).

## Goals / Non-Goals

**Goals:**
- Fijar un conjunto de 20 funciones (Python + JS/TS) con lista de casos
  documentada, listo para que Fase 2 lo consuma sin ambigüedad.
- Que el formato del manifest permita comparar automáticamente
  generado-vs-documentado (soporta "conviene automatizar la corrida con un
  script desde el inicio", 04-plan.md).
- Que la variedad de forma entre las 20 funciones sea intencional, no
  incidental, para que medir cobertura en Fase 2 sea informativo.
- Que los fragmentos "difíciles de probar" ejerciten RF-09 en los ángulos que
  importan, incluido uno que el plan original no cubría (dependencia de
  red/DB).

**Non-Goals:**
- No se corre el conjunto contra el modelo todavía — eso es Fase 2.
- No se escribe el script de automatización de scoring — también Fase 2.
- No se decide `effort` por defecto ni se mide costo real — Fase 2.
- No se valida que las 20 funciones sean "representativas" de código real de
  estudiantes vía muestreo o encuesta — quedan inventadas por el equipo por
  decisión explícita (ver Decisiones).

## Decisions

### 1. Tamaño: 20 funciones en total, no 20 + 3-4 aparte

`04-plan.md` dice textual "incluir a propósito **en ese conjunto**" para los
difíciles. Se toma esa lectura como autoritativa sobre la redacción de
`sprint-01.md`, que los lista como dos checkboxes separados de forma ambigua.

**Alternativa considerada:** 24 en total (20 normales + 4 aparte). Se
descarta porque rompe los denominadores de PRD §9 (`≥15/20`, `≥17/20`) — o
bien el conteo real deja de ser 20 y esos criterios habría que reabrirlos, lo
cual no está pedido en ningún documento fuente.

### 2. Reparto de idioma: 10 Python / 10 JavaScript-TypeScript

Reparto parejo por defecto. El PRD no da ninguna razón de negocio para
desbalancear, y ambos lenguajes son igual de centrales al MVP (PRD §5).

**Alternativa considerada:** reparto desbalanceado (ej. más Python si el
equipo lo domina más). Se descarta por falta de justificación: el objetivo es
evidencia igual de sólida para calibrar `effort` en los dos lenguajes, no
optimizar la velocidad de redacción del conjunto.

### 3. Manifest estructurado (YAML/JSON) en vez de tabla en prosa

Cada caso documentado lleva: `tipo` (`camino_feliz` | `valor_limite` |
`entrada_invalida`), `entrada` y `esperado` (salida o comportamiento
esperado), explícitos y parseables.

**Alternativa considerada:** tabla markdown en prosa, más coherente con el
resto de `docs/` (todo en Markdown). Se descarta porque Fase 2 compara 20
funciones × 3 niveles de `effort` (más el pase opcional de Opus 5) — juzgar
cobertura releyendo prosa a mano no escala y es subjetivo. El costo de
escribir el manifest ahora se paga una vez; el costo de interpretar prosa se
pagaría repetido en cada corrida de Fase 2.

### 4. Matriz de atributos fijada antes de escribir las funciones

Se define de antemano qué combinación de atributos deben cubrir las 20
funciones en conjunto: tipo de retorno (escalar / colección / `None`),
manejo de error (excepción vs. código de retorno), parámetros con valor por
defecto, y al menos una función recursiva.

**Alternativa considerada:** dejar que la variedad emerja naturalmente al
escribir las funciones. Se descarta por riesgo concreto: sin matriz, es fácil
terminar con 20 variaciones de "valida input y retorna bool", donde "borde" e
"inválido" significan casi lo mismo en las 20 — cumple el número 20 en el
papel pero mide poco en Fase 2.

### 5. Difíciles: los 3 fijos del plan + dependencia de red/DB como 4to

Se agrega un 4to fragmento difícil no listado en `04-plan.md`: una función
con dependencia de red o base de datos. Motivo: el PRD excluye del MVP
"generación de mocks para dependencias externas complejas" (§5) — sin un caso
así en el conjunto, RF-09 nunca se ejercita contra ese ángulo específico, solo
contra los tres que sí anticipó el plan (script sin funciones, hora del
sistema, lectura de archivo).

**Alternativa considerada:** ceñirse estrictamente a los 3 fijos del plan sin
agregar ninguno. Se descarta porque deja sin cubrir un modo de falla que el
propio equipo ya identificó como relevante en
[05-proveedor-llm.md](../../../docs/costos/05-proveedor-llm.md).

### 6. Origen de las funciones normales: inventadas por el equipo

**Alternativa considerada:** adaptar tareas reales de estudiantes de FIME
(anonimizadas). Más representativo del usuario real, pero se descarta para
este conjunto porque exige trámite de permiso/anonimización que no cabe en un
sprint de 2 semanas, y porque tareas reales no necesariamente traen a
propósito la variedad de forma que exige la Decisión 4.

### 7. Ubicación: `docs/evaluacion/`

Sigue la convención ya establecida del repo: `docs/` organizado por carpetas
temáticas (`producto/`, `arquitectura/`, `costos/`, `gestion/`, `sprints/`).

**Alternativa considerada:** `eval-set/` en la raíz del repo, más cerca de
donde Fase 2 pondrá el script de automatización. Se descarta por romper la
convención de que todo contenido no-código vive bajo `docs/`; si Fase 2
necesita un script, ese script puede vivir fuera de `docs/` sin que el
manifest se mueva con él.

## Risks / Trade-offs

- **El manifest estructurado es más lento de escribir a mano que prosa** →
  Mitigación: el costo se paga una sola vez en Sprint 1; se amortiza en cada
  corrida de Fase 2 (3 niveles de `effort` × 20 funciones, más el pase
  opcional de Opus 5).
- **La matriz de atributos puede forzar funciones artificiales que no se
  sienten como código real de estudiante** → Mitigación: la matriz fija
  atributos (tipo de retorno, manejo de error, etc.), no la redacción; cada
  función individual se sigue escribiendo con el estilo de ~50 líneas que
  describe el PRD (sección 1).
- **Inventar las funciones en vez de usar código real reduce el realismo del
  conjunto** → Mitigación aceptada explícitamente (Decisión 6): el trade-off
  es velocidad de entrega en Sprint 1 contra representatividad; se puede
  revisar en una fase posterior si Fase 2 revela que el conjunto no
  discrimina bien entre niveles de `effort`.
- **Agregar el 4to difícil (red/DB) no estaba en el plan original** →
  Mitigación: esta propuesta lo declara y justifica explícitamente (Decisión
  5) en vez de agregarlo en silencio; si el equipo lo rechaza en revisión,
  el conjunto vuelve a 3 fijos sin romper el resto del diseño.

## Migration Plan

No aplica migración de datos ni de código — es la primera versión del
conjunto de evaluación, no reemplaza nada existente. Pasos de entrega:

1. Aplicar esta change (`tasks.md`) para producir el manifest bajo
   `docs/evaluacion/` y corregir `sprint-01.md`.
2. Revisión por al menos otra persona del equipo (Definition of Done,
   [07-metodologia.md](../../../docs/gestion/07-metodologia.md) sección 6).
3. Marcar el punto del Sprint Backlog como hecho en `sprint-01.md`.
4. Fase 2 consume `docs/evaluacion/` tal cual — no requiere cambios en este
   conjunto salvo que la calibración revele un problema con el diseño del
   manifest o la matriz de atributos.

## Open Questions

- ¿Quién del equipo escribe las 20 funciones? `07-metodologia.md` asigna el
  conjunto de evaluación al área "LLM y prompt", que sigue sin persona
  asignada más allá del Product Owner. No bloquea esta propuesta, sí bloquea
  la ejecución de `tasks.md`.
- Si Fase 2 revela que el conjunto no discrimina bien entre niveles de
  `effort` (todas las funciones pasan igual en `low` que en `high`), ¿se
  revisa el conjunto o se acepta el resultado como válido? Se deja abierto
  para decidirse con datos reales de Fase 2, no de antemano.

## Why

La Fase 2 (Calibración) no puede medir `effort` sobre Sonnet 5 ni los criterios
de éxito de [01-prd.md](../../../docs/producto/01-prd.md) sección 9 (cobertura
≥15/20, validez ≥17/20, cero tautológicas) sin un conjunto de evaluación fijo
contra el cual comparar. Es el primer punto pendiente del Sprint 1
([sprint-01.md](../../../docs/sprints/sprint-01.md)) y bloquea toda la Fase 2 —
por eso va antes que cualquier código de producto.

El punto tal como está escrito en el sprint backlog es ambiguo en un punto
crítico: lista "20 funciones" y "3-4 fragmentos difíciles" como dos renglones
separados, pero [04-plan.md](../../../docs/gestion/04-plan.md) dice
textualmente "incluir a propósito **en ese conjunto**" — los difíciles van
dentro de las 20, no aparte. Esta propuesta fija esa lectura antes de que se
arme el set, para que los denominadores de PRD §9 (`/20`) sigan siendo
correctos.

## What Changes

- Se crea el conjunto de evaluación de Testly: **20 funciones en total**
  (16-17 "normales" + 3-4 "difíciles de probar" incluidas en el mismo
  conteo), repartidas 10 Python / 10 JavaScript-TypeScript.
- Cada función se documenta con su lista de casos esperados (camino feliz,
  valor límite, entrada inválida) en un **manifest estructurado** (entrada +
  salida/comportamiento esperado explícito por caso), para que Fase 2 pueda
  comparar generado-vs-documentado con un script en vez de a ojo.
- Se fija de antemano una **matriz de atributos** que las 20 funciones deben
  cubrir en conjunto (tipo de retorno, manejo de error, args con default,
  recursión), para que los casos borde/inválido no se repitan de forma
  redundante entre funciones.
- Se fija la composición exacta de los 3-4 fragmentos difíciles: script sin
  funciones, dependencia de la hora del sistema, lectura de archivo, y
  dependencia de red/base de datos — este último no estaba en el plan
  original, se agrega para ejercitar RF-09 contra el caso "fuera del MVP:
  mocks de dependencias externas complejas" (PRD §5).
- Todo el conjunto queda documentado bajo `docs/evaluacion/`.
- Se corrige la redacción de [sprint-01.md](../../../docs/sprints/sprint-01.md)
  para eliminar la ambigüedad de 20 vs 24.

Este cambio no toca código de producto: la Fase 1 (`/api/generar`) no ha
arrancado y no existe `src/` en el repo. Es un entregable de documentación /
dataset de la Fase 0.

## Capabilities

### New Capabilities
- `conjunto-evaluacion`: define la estructura del manifest de evaluación, la
  matriz de atributos obligatoria, la composición exacta de las 20 funciones
  (normales + difíciles) y dónde vive el conjunto en el repo. Es la
  especificación que Fase 2 usa para automatizar la corrida y el scoring.

### Modified Capabilities
(ninguna — no hay specs previas en `openspec/specs/`, este es el primer
capability del proyecto)

## Impact

- **Archivos nuevos:** `docs/evaluacion/` (manifest de las 20 funciones +
  casos documentados + matriz de atributos).
- **Archivos modificados:** `docs/sprints/sprint-01.md` (desambigua el
  checkbox de 20 vs 24 funciones).
- **Dependencias hacia adelante:** Fase 2
  ([04-plan.md](../../../docs/gestion/04-plan.md)) consume este manifest
  directo para automatizar la corrida contra Sonnet 5 en `low`/`medium`/`high`
  y para el pase opcional de Opus 5.
- **Sin impacto en código de producto:** no existe `src/` todavía; esto es
  puramente documentación/dataset de Fase 0.

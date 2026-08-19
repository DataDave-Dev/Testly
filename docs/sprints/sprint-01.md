# Sprint 1 — Preparación (Fase 0) — Testly

Fecha: 2026-08-19
Versión: 0.2
Estado: por iniciar (fechas de inicio/fin se fijan en Sprint Planning)

## Objetivo del sprint

Cerrar equipo, presupuesto y conjunto de evaluación antes de que arranque la
Fase 1. El conjunto de evaluación va primero a propósito: sin él no se puede
calibrar el modelo ni el nivel de `effort` en la Fase 2, y esas dos
decisiones definen el costo del proyecto entero (ver
[04-plan.md](../gestion/04-plan.md), Fase 0).

## Sprint Backlog

- [x] Confirmar equipo y reparto de responsabilidades — Product Owner: Osmar
  Emilio Solís Olivares; Scrum Master: Isela Esmeralda Guerrero Martínez
  ([07-metodologia.md](../gestion/07-metodologia.md) sección 2)
- [x] Presupuesto — **resuelto**: $20 USD ya disponibles
  ([03-costos.md](../costos/03-costos.md))
- [x] Límite de cuota por usuario — **resuelto**: 10 cada 30 días con sesión,
  3/día por IP sin sesión
- [x] Armar el conjunto de evaluación: **20 funciones en total** (10 Python /
  10 JavaScript-TypeScript), de las cuales 4 son fragmentos **difíciles de
  probar** incluidos dentro del mismo conteo de 20, no aparte (script sin
  funciones, depende de la hora del sistema, lee un archivo, dependencia de
  red/base de datos) — verifica RF-07, RF-08, RF-09 y los criterios de éxito
  de [01-prd.md](../producto/01-prd.md) sección 9. Cada función normal trae
  su lista documentada de casos (camino feliz, valor límite, entrada
  inválida) en `docs/evaluacion/`. Ver
  [change armar-conjunto-evaluacion](../../openspec/changes/armar-conjunto-evaluacion/)

## Entregable

Conjunto de evaluación completo y presupuesto confirmado. Coincide con el
Sprint Review de este sprint (ver [04-plan.md](../gestion/04-plan.md), Fase 0).

## Fuera de este sprint

- Cualquier RF-01…RF-24 de producto — arrancan en la Fase 1
- Infra de despliegue en Cloudflare Workers — es tarea de la Fase 1, no de la
  Fase 0
- Diseño de interfaz — Fase 4

## Definition of Done

Aplica la definición general del equipo
([07-metodologia.md](../gestion/07-metodologia.md) sección 6): revisión por al menos
otra persona, pruebas unitarias si la tarea toca `src/lib/` (no aplica a este
sprint), y si una tarea cambia una decisión ya documentada, el doc
correspondiente en `docs/` se actualiza en el mismo sprint.

## Historial de versiones

| Versión | Fecha | Cambio |
|---|---|---|
| 0.1 | 2026-08-19 | Primera versión, contenido tomado de la Fase 0 de [04-plan.md](../gestion/04-plan.md) |
| 0.2 | 2026-08-19 | Conjunto de evaluación armado y marcado como hecho: 20 funciones en total (10 Python / 10 JS-TS), difíciles incluidos en el conteo, no aparte. Desambigua la redacción original que los listaba como dos puntos separados. Ver `docs/evaluacion/` y [change armar-conjunto-evaluacion](../../openspec/changes/armar-conjunto-evaluacion/) |

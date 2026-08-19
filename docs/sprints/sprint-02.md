# Sprint 2 — Núcleo de la generación (1/2) — Testly

Fecha: 2026-08-19
Versión: 0.2
Estado: por iniciar
Inicio: 2026-09-02 · Fin: 2026-09-15

## Objetivo del sprint

Dejar el pipeline de generación funcionando de punta a punta: proyecto Astro
desplegado en Cloudflare, streaming probado en producción (no solo en
`localhost`), extracción y validación del bloque generado, y una pantalla
mínima para pegar código y descargar el resultado. Es el punto de mayor
riesgo técnico del proyecto — si esto no funciona, el resto sobra (ver
[04-plan.md](../gestion/04-plan.md), Fase 1).

La protección del endpoint (Rate Limiting + Turnstile + `anon_quota`) y el
sandbox de ejecución (RF-25) quedan para el Sprint 3: cierran la Fase 1
justo antes de que la Fase 2 reuse el sandbox.

## Sprint Backlog

- [ ] Proyecto Astro con adapter de Cloudflare (`@astrojs/cloudflare`)
- [ ] Reclutar variables de entorno para local y producción: `LLM_API_KEY`,
  `LLM_MODEL`, `LLM_EFFORT`, `MAX_CODE_CHARS`, `LLM_MAX_OUTPUT_TOKENS`,
  `MAINTENANCE_MODE`, `MAINTENANCE_BYPASS_SECRET`, `LAUNCH_DATE` (ver
  [02-arquitectura.md](../arquitectura/02-arquitectura.md) sección 8).
  `.env` local y secrets/vars del proyecto en Cloudflare Workers; el servidor
  falla al arrancar si falta alguna
- [ ] Modo mantenimiento con contador: middleware que muestra
  `/en-construccion` (fecha de lanzamiento + contador) a cualquier visitante
  mientras `MAINTENANCE_MODE=true`, con bypass por cookie/query param para
  el equipo (ver [02-arquitectura.md](../arquitectura/02-arquitectura.md)
  sección 2.15). Necesario porque este sprint ya despliega a producción
- [ ] Prueba de streaming largo desplegada: endpoint que transmita 60
  segundos y llegue completo al navegador desde el dominio de Cloudflare, no
  desde `localhost`
- [ ] Aserciones de validación por framework
  ([02-arquitectura.md](../arquitectura/02-arquitectura.md) sección 2.11),
  con sus pruebas unitarias
- [ ] `frameworks.ts` y `extraerTest.ts` con sus pruebas unitarias
- [ ] Primera versión del prompt de sistema
- [ ] Endpoint `/api/generar` funcionando de extremo a extremo, sin sesión
- [ ] Pantalla mínima: pegar código, elegir lenguaje y framework, ver el
  resultado en streaming, descargar el archivo

## Entregable

Un archivo de pruebas real, generado desde el navegador en producción y
descargado. Coincide parcialmente con el entregable de Fase 1 completa (ver
[04-plan.md](../gestion/04-plan.md)); la protección del endpoint y el
sandbox de ejecución cierran en el Sprint 3.

## Fuera de este sprint

- Rate Limiting, Turnstile y tabla `anon_quota` — Sprint 3
- Sandbox de ejecución del lado del cliente (RF-25) — Sprint 3
- Cuentas, persistencia (Turso, Better Auth) — Fase 3
- Diseño de interfaz — Fase 4

## Definition of Done

Aplica la definición general del equipo
([07-metodologia.md](../gestion/07-metodologia.md) sección 6): revisión por
al menos otra persona, pruebas unitarias para lo que toque `src/lib/`, y si
una tarea cambia una decisión ya documentada, el doc correspondiente en
`docs/` se actualiza en el mismo sprint.

## Historial de versiones

| Versión | Fecha | Cambio |
|---|---|---|
| 0.1 | 2026-08-19 | Primera versión. Backlog tomado de la mitad 1/2 de la Fase 1 de [04-plan.md](../gestion/04-plan.md), más el reclutamiento de variables de entorno para local y producción |
| 0.2 | 2026-08-19 | Se agrega modo mantenimiento con contador (página "en construcción" mientras el sitio no es público), necesario porque este sprint ya despliega a producción. Ver [02-arquitectura.md](../arquitectura/02-arquitectura.md) sección 2.15 |

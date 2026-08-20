## Why

El Sprint 2 ([sprint-02.md](../../../docs/sprints/sprint-02.md)) arranca la
Fase 1 — el punto de mayor riesgo técnico del proyecto. Todo lo demás del
sprint (variables de entorno en Cloudflare, modo mantenimiento, prueba de
streaming desplegada, endpoint `/api/generar`, pantalla mínima) necesita un
proyecto Astro con el adapter de Cloudflare ya instalado y compilando: es la
base sobre la que corre el resto. Sin este primer paso no hay dónde poner
ningún endpoint ni ninguna página.

Esta propuesta cubre **exclusivamente** el primer renglón del Sprint Backlog:
"Proyecto Astro con adapter de Cloudflare (`@astrojs/cloudflare`)". Los demás
puntos del sprint son entregables separados, fuera de esta change.

## What Changes

- Se inicializa un proyecto Astro en la raíz del repo (sin monorepo — ver
  [02-arquitectura.md](../../../docs/arquitectura/02-arquitectura.md) sección
  2.10), con TypeScript en modo estricto.
- Se instala y configura el adapter `@astrojs/cloudflare`: `output: 'server'`
  en `astro.config.mjs`, requisito documentado en la sección 2.2 de
  arquitectura ("Lo que hay que verificar temprano").
- Se agrega configuración de Wrangler (`wrangler.jsonc` o `wrangler.toml`)
  necesaria para que el adapter funcione en local, apuntando a Cloudflare
  **Workers** (no Pages) — la elección de hosting de la sección 2.8.
- Se deja el build (`pnpm run build`) y el arranque en local (`pnpm run dev`)
  verificados y sin errores.
- Se actualiza `.gitignore` para los artefactos que este scaffold introduce
  (`node_modules/`, `dist/`, `.astro/`, `.wrangler/`).

**No incluye** (confirmado con el usuario, ver `design.md` § Non-Goals): el
primer deploy real a Cloudflare, CI/CD, React/islas, Tailwind, la estructura
completa de carpetas de `06-estructura-carpetas.md`, variables de entorno de
producto, ni el modo mantenimiento. Todos son puntos separados del mismo
sprint o de sprints posteriores.

## Capabilities

### New Capabilities
- `proyecto-astro-cloudflare`: define los requisitos del scaffold inicial —
  proyecto Astro en la raíz del repo, adapter de Cloudflare configurado en
  modo SSR, build y arranque local verificados, sin deploy.

### Modified Capabilities
(ninguna — no hay specs previas relacionadas en `openspec/specs/`; la única
existente, `conjunto-evaluacion`, es un capability de datos/documentación sin
relación con el código de producto)

## Impact

- **Archivos nuevos:** proyecto Astro completo en la raíz del repo
  (`package.json`, `astro.config.mjs`, `tsconfig.json`, `wrangler.jsonc`,
  `pnpm-lock.yaml`, `pnpm-workspace.yaml`, `src/pages/index.astro` mínimo
  generado por el scaffolding).
- **Archivos modificados:** `.gitignore` (agrega `node_modules/`, `dist/`,
  `.astro/`, `.wrangler/`).
- **Package manager: pnpm, no npm.** Decisión tomada durante la
  implementación (el usuario pidió el cambio al intentar correr el proyecto)
  — ver `design.md` § Decisiones, punto 5.
- **Dependencias hacia adelante:** todo punto restante del Sprint 2 (env
  vars, modo mantenimiento, prueba de streaming, validación, prompt,
  endpoint, pantalla mínima) asume que este scaffold ya existe y compila.
- **Impacto en `docs/`:** [02-arquitectura.md](../../../docs/arquitectura/02-arquitectura.md)
  sección 2.10 mencionaba `npm run dev` como evidencia de que no hace falta
  Docker para desarrollo; se actualizó a `pnpm dev` para reflejar el cambio
  de package manager (Definition of Done,
  [07-metodologia.md](../../../docs/gestion/07-metodologia.md) sección 6).

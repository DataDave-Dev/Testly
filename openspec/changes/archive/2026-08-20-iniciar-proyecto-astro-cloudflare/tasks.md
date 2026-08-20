## 1. Scaffold del proyecto Astro

- [x] 1.1 Inicializar Astro en la raíz del repo (`pnpm create astro@latest .`
      o equivalente), template TypeScript estricto, sin instalar
      integraciones de UI todavía
- [x] 1.2 Verificar que `tsconfig.json` extiende el preset `strict` (o
      superior) de `astro/tsconfigs/`
- [x] 1.3 Confirmar `package.json` en la raíz, sin campo `workspaces`

## 2. Adapter de Cloudflare

- [x] 2.1 Instalar `@astrojs/cloudflare` (`npx astro add cloudflare` o
      instalación manual)
- [x] 2.2 Configurar `astro.config.mjs`/`.ts`: `output: 'server'` y
      `adapter: cloudflare()`
- [x] 2.3 Crear/ajustar `wrangler.jsonc` o `wrangler.toml`: target Cloudflare
      Workers (no Pages), `compatibility_date` explícito, `nodejs_compat` si
      el adapter lo requiere

## 3. Verificación local

- [x] 3.1 Ejecutar `pnpm run build` y confirmar salida sin errores,
      `dist/` generado
- [x] 3.2 Ejecutar `pnpm run dev` y confirmar que la página por defecto sirve
      sin excepciones en consola
- [x] 3.3 (Opcional, si el equipo ya tiene Wrangler CLI) `wrangler dev` para
      validar el proyecto contra el runtime local de Workers, sin desplegar

## 4. Limpieza y cierre

- [x] 4.0 Cambiar package manager de npm a pnpm (pedido por el usuario al
      probar el proyecto): borrar `node_modules/`/`package-lock.json`,
      `pnpm install`, `pnpm-workspace.yaml` con `onlyBuiltDependencies`,
      `allowBuilds` y `minimumReleaseAge: 0` (scopeado al repo, ver
      `design.md` Decisión 5). Actualizar `02-arquitectura.md` sección 2.10
      de `npm run dev` a `pnpm dev`. Re-verificar `pnpm run build` y
      `pnpm run dev`
- [x] 4.1 Actualizar `.gitignore`: agregar `node_modules/`, `dist/`,
      `.astro/`, `.wrangler/`
- [x] 4.2 Confirmar que ningún punto posterior del sprint quedó adelantado
      (sin React, sin Tailwind, sin deploy, sin estructura completa de
      `06-estructura-carpetas.md`) — alcance limitado al renglón 1 del
      Sprint Backlog. Verificado: `package.json` solo trae `astro`,
      `@astrojs/cloudflare`, `wrangler`; no se corrió `wrangler deploy`
- [x] 4.3 Revisión por al menos otra persona del equipo (Definition of Done,
      [07-metodologia.md](../../../docs/gestion/07-metodologia.md) sección 6)
- [x] 4.4 Marcar el renglón "Proyecto Astro con adapter de Cloudflare" como
      hecho en [sprint-02.md](../../../docs/sprints/sprint-02.md)

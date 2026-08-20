## Context

El repo no tiene código de producto todavía (no existe `src/`, ni
`package.json`). `02-arquitectura.md` ya decidió el stack completo — este
diseño no re-decide nada, solo determina cómo ejecutar la primera pieza:
Astro con el adapter de Cloudflare, compilando en local.

El resto del Sprint 2 depende de que este scaffold exista: el endpoint de
streaming, la prueba de 60 segundos desplegada, el modo mantenimiento y la
pantalla mínima se construyen todos sobre este proyecto Astro.

## Goals / Non-Goals

**Goals:**
- Proyecto Astro inicializado en la raíz del repo, TypeScript estricto.
- Adapter `@astrojs/cloudflare` instalado y configurado en modo SSR
  (`output: 'server'`).
- Configuración de Wrangler presente y válida para el runtime de Cloudflare
  Workers (compatibilidad Node vía `nodejs_compat`, `compatibility_date`
  vigente).
- `pnpm run build` y `pnpm run dev` funcionando sin errores en local.

**Non-Goals:**
- No se hace ningún deploy real a Cloudflare. La primera URL viva llega con
  el punto "prueba de streaming largo desplegada" del mismo sprint
  ([sprint-02.md](../../../docs/sprints/sprint-02.md)) — confirmado
  explícitamente con el usuario.
- No se configura CI/CD de ningún tipo (ni GitHub Actions ni deploy manual
  documentado) — por lo mismo, no hay deploy en esta change.
- No se instala `@astrojs/react` ni se crean islas — llega con el punto de
  "pantalla mínima".
- No se instala ni configura Tailwind — arquitectura sección 2.7 lo ubica en
  Fase 4, no en Fase 1.
- No se construye la estructura completa de carpetas de
  [06-estructura-carpetas.md](../../../docs/arquitectura/06-estructura-carpetas.md)
  (`middleware.ts`, `pages/api/*`, `components/*`, `lib/*`). Esta change deja
  solo lo que el scaffolding de Astro genera por defecto más el adapter; los
  puntos posteriores del sprint van poblando esa estructura conforme cada
  pieza se construye.
- No se tocan variables de entorno de producto (`LLM_API_KEY`, etc.) — es el
  punto 2 del sprint, separado.

## Decisions

### 1. El proyecto Astro vive en la raíz del repo, no en una subcarpeta

`02-arquitectura.md` sección 2.10 descarta monorepo/workspaces explícitamente
("Es un proyecto"). Inicializar Astro en la raíz evita inventar una capa de
workspace que el resto de la documentación no anticipa.

**Alternativa considerada:** subcarpeta `app/` o `web/` con el proyecto
Astro dentro, dejando la raíz solo para `docs/`. Se descarta: ningún
documento de arquitectura la menciona, y todos los ejemplos de código de
`02-arquitectura.md` (rutas como `src/lib/db/generations.ts`) asumen `src/`
en la raíz.

### 2. Cloudflare Workers como target, no Cloudflare Pages

El adapter `@astrojs/cloudflare` puede apuntar a Pages o a Workers. La
arquitectura (sección 2.8) elige explícitamente Workers por el límite de CPU
en vez de tiempo de pared, y describe los planes gratis/$5 en términos de
Workers. Pages Functions comparte el mismo límite de CPU pero el documento
fuente nombra Workers de forma consistente en toda la sección.

**Alternativa considerada:** Cloudflare Pages, que también soporta SSR con
el mismo adapter y tiene flujo de deploy más simple vía Git. Se descarta por
seguir la decisión ya escrita en `02-arquitectura.md`, no por una limitación
técnica nueva encontrada aquí.

### 3. Sin deploy en esta change

Confirmado con el usuario (dos preguntas, ver Impact en `proposal.md`): este
punto llega solo hasta build local verificado. El primer deploy real ocurre
en el punto "prueba de streaming largo desplegada", que además necesita un
endpoint real para tener sentido — desplegar un scaffold vacío no verifica
nada que ese punto no vaya a verificar de todas formas.

**Alternativa considerada:** incluir un primer deploy de "hola mundo" aquí
para separar "¿el pipeline de deploy funciona?" de "¿el streaming funciona
en producción?". Se descarta por decisión explícita del usuario — mantener
esta change estrictamente al alcance del renglón 1 del sprint backlog, sin
adelantar trabajo de renglones posteriores.

### 4. Estructura mínima, no la estructura completa de `06-estructura-carpetas.md`

Aunque la opción A ya está decidida, crear carpetas vacías (`components/`,
`lib/db/`, etc.) sin archivos reales no aporta nada verificable — no hay
ningún requisito SHALL que se pueda probar sobre una carpeta vacía. Cada
punto posterior del sprint crea su propia porción de esa estructura junto
con el código real que la llena.

**Alternativa considerada:** crear el árbol completo de carpetas con
archivos `.gitkeep` como parte de este scaffold. Se descarta por YAGNI: son
carpetas sin contenido que no verifican nada hasta que el código que les
corresponde exista.

### 5. pnpm como package manager, no npm

El scaffold inicial se hizo con npm (única herramienta mencionada en
`02-arquitectura.md` al momento de escribir esta change). Al intentar correr
el proyecto, el usuario ya tenía pnpm como herramienta de trabajo y pidió
cambiar — decisión del usuario, no una limitación técnica de npm.

Cambio aplicado: se borró `node_modules/` y `package-lock.json`, se corrió
`pnpm install` para generar `pnpm-lock.yaml`, y se agregó
`pnpm-workspace.yaml` (requerido desde pnpm 10+ para settings que antes
vivían en el campo `pnpm` de `package.json`, como `onlyBuiltDependencies`).
`02-arquitectura.md` sección 2.10 se actualizó de `npm run dev` a `pnpm dev`.

**Nota sobre `pnpm-workspace.yaml`:** el nombre del archivo sugiere monorepo,
pero acá solo contiene configuración de pnpm (`onlyBuiltDependencies`,
`minimumReleaseAge`, `allowBuilds`) para un único paquete. No contradice la
Decisión 1 (sin monorepo/workspaces) — pnpm simplemente reusa ese archivo
como home de su configuración general desde la v10.

**Dos bloqueos de pnpm que hubo que resolver, ambos scopeados a este
proyecto (no a la config global del usuario):**

- **`ERR_PNPM_IGNORED_BUILDS`**: pnpm no corre scripts `postinstall` de
  dependencias por default. `esbuild` y `workerd` los necesitan para bajar
  su binario nativo. Se permitieron explícitamente en `pnpm-workspace.yaml`
  (`onlyBuiltDependencies` + `allowBuilds: { esbuild: true, workerd: true }`)
  — allowlist explícita y commiteada, no aprobación interactiva silenciosa.
- **`ERR_PNPM_MINIMUM_RELEASE_AGE_VIOLATION`**: política anti supply-chain
  de pnpm que rechaza paquetes publicados muy recientemente. Bloqueaba
  `astro`, `@astrojs/cloudflare` y sus transitivas. Se relajó con
  `minimumReleaseAge: 0` en `pnpm-workspace.yaml`, **scopeado a este repo**
  (no se tocó la config global de pnpm del usuario) — decisión confirmada
  con el usuario antes de aplicarse, dado que es una política de seguridad.

**Alternativa considerada:** quedarse en npm, que no tiene ninguna de estas
dos políticas por default. Se descarta porque el usuario pidió explícitamente
el cambio a pnpm.

## Risks / Trade-offs

- **`minimumReleaseAge: 0` reduce la protección de pnpm contra ataques de
  supply-chain vía paquetes recién publicados/comprometidos** → Mitigación:
  el repo sigue teniendo `package.json`/`pnpm-lock.yaml` versionados y
  revisables en cada PR (Definition of Done exige revisión por otra
  persona); si el equipo prefiere más margen, subir el valor (en horas) es
  un cambio de una línea en `pnpm-workspace.yaml`.
- **El adapter de Cloudflare puede no ser 100% compatible con el runtime de
  Workers en algún paquete** (riesgo ya anotado en
  [04-plan.md](../../../docs/gestion/04-plan.md): "El equipo no domina Astro
  con SSR") → Mitigación: este scaffold es exactamente el punto donde ese
  riesgo se detecta temprano, con semanas de margen antes de que dependa de
  él código real.
- **No desplegar en esta change retrasa la primera señal de "funciona en
  Cloudflare de verdad"** hasta el siguiente punto del sprint → Mitigación
  aceptada explícitamente por el usuario; el build local ya verifica que el
  adapter genera output válido para el runtime de Workers antes de intentar
  cualquier deploy.
- **Wrangler requiere una versión y una configuración (`compatibility_date`)
  que puede quedar desactualizada** si el sprint tarda en llegar al deploy →
  Mitigación: `compatibility_date` se fija a la fecha de esta change: no
  bloquea el build local y se puede subir sin fricción cuando llegue el
  punto de deploy real.

## Migration Plan

No aplica migración de datos — es el primer código de producto del repo.
Pasos de entrega:

1. Aplicar esta change (`tasks.md`) para producir el scaffold de Astro +
   adapter en la raíz del repo.
2. Verificar `pnpm run build` y `pnpm run dev` localmente (criterio de
   aceptación de la spec).
3. Revisión por al menos otra persona del equipo (Definition of Done,
   [07-metodologia.md](../../../docs/gestion/07-metodologia.md) sección 6).
4. Marcar el renglón correspondiente en `sprint-02.md` como hecho.
5. Los puntos siguientes del Sprint 2 (env vars, modo mantenimiento, prueba
   de streaming desplegada, etc.) consumen este scaffold directamente.

## Open Questions

- Versión exacta de Node objetivo para Wrangler/Cloudflare — no hay archivo
  `.nvmrc` ni `engines` en el repo todavía. No bloquea esta change (el
  scaffolding de Astro trae su propia versión mínima soportada); se puede
  fijar explícitamente si algún punto posterior del sprint lo necesita.
- Nombre final del Worker en `wrangler.jsonc`/`wrangler.toml` (afecta la URL
  de producción) — no importa para el build local de esta change; se decide
  junto con el primer deploy real.

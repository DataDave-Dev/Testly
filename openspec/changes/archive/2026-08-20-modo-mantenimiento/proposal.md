## Why

El proyecto Astro solo tiene el scaffold por defecto (`src/pages/index.astro`
con un `<h1>Astro</h1>`). El equipo tiene un boceto de diseño
(`docs/diseno/modo-mantenimiento/modo-mantenimiento.dc.html`, hecho con
herramienta de diseño externa) para una pantalla de "modo mantenimiento" /
coming-soon que debe ser la única vista pública mientras el resto del
producto (Sprint 2+) no existe. `proyecto-astro-cloudflare` ya menciona el
modo mantenimiento como trabajo pendiente sobre esa base.

## What Changes

- Reemplazar el contenido de `src/pages/index.astro` por la vista de modo
  mantenimiento, traducida del boceto `.dc.html` a Astro real (sin el
  andamiaje de la herramienta de diseño: `x-dc`, `support.js`,
  `data-dc-script`, clase `DCLogic`, placeholders `{{}}`).
- Extraer el markup a `src/components/maintenance/MaintenanceView.astro`
  (estructura A ya decidida en `docs/arquitectura/06-estructura-carpetas.md`).
- Instalar y configurar Tailwind CSS v4 (`tailwindcss` + `@tailwindcss/vite`
  en `astro.config.mjs`) y reemplazar los estilos inline del boceto por
  clases de Tailwind. Tokens de diseño (colores oklch, fuentes, spacing
  clamp) en `src/styles/tokens.css`.
- Instalar `gsap` y usarlo para:
  - las dos bandas de marquee (reemplazando los `@keyframes` CSS del
    boceto por timelines infinitos),
  - una animación de entrada con stagger al cargar la página,
  - una animación de "flip" por dígito cuando cambia el valor del
    countdown.
- Countdown apunta a `2026-12-01T00:00:00` (fecha límite real del proyecto,
  `docs/gestion/08-limites.md`), no a la fecha placeholder del boceto
  (`2026-09-15`).
- No se agrega mecanismo de activar/desactivar modo mantenimiento
  (middleware, flag, env var): por ahora es la única página del sitio, no
  hay nada que interceptar.

## Capabilities

### New Capabilities
- `vista-modo-mantenimiento`: página pública única del sitio mientras no
  existe el resto del producto — countdown a la fecha límite, banda de
  marquee animada, sin mecanismo de activación/desactivación.

### Modified Capabilities
(ninguna — `proyecto-astro-cloudflare` no cambia sus requisitos, solo se
construye sobre esa base)

## Impact

- Código: `src/pages/index.astro` (reescrito), nuevos
  `src/components/maintenance/MaintenanceView.astro` y
  `src/styles/tokens.css`.
- Config: `astro.config.mjs` (plugin de Tailwind), `package.json`
  (dependencias nuevas: `tailwindcss`, `@tailwindcss/vite`, `gsap`).
- Sin impacto en backend, API ni base de datos — el proyecto todavía no
  tiene ninguno de esos.

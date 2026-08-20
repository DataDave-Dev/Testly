## Context

`src/pages/index.astro` es hoy el scaffold por defecto de Astro. El único
material de diseño disponible es
`docs/diseno/modo-mantenimiento/modo-mantenimiento.dc.html`: un boceto
producido por una herramienta de diseño visual (wrapper `x-dc`,
`support.js`, `data-dc-script`, clase `DCLogic` con `{{placeholders}}`). No
es código de producción — hay que traducirlo a Astro real.

El proyecto ya tiene convenciones fijadas que aplican directo:
- Estructura de carpetas por tipo (`docs/arquitectura/06-estructura-carpetas.md`,
  opción A): `pages/`, `components/`, `lib/`, `styles/`.
- Cultura anti-sobreingeniería explícita en `AGENTS.md`: sin abstracciones
  para casos hipotéticos, dependencias solo con autorización explícita
  (dada aquí para Tailwind y GSAP).
- Fecha límite real del proyecto: `2026-12-01` (`docs/gestion/08-limites.md`),
  distinta a la fecha placeholder del boceto (`2026-09-15`).

## Goals / Non-Goals

**Goals:**
- Traducir el boceto a una página Astro real, sin dependencias del tool de
  diseño (`x-dc`, `support.js`, `DCLogic`, `{{}}`).
- Countdown correcto y con misma hora para cualquier visitante,
  independientemente de su zona horaria.
- Tailwind v4 y GSAP configurados y funcionando en Astro 7 (SSR/Cloudflare).
- Componente separado (`components/maintenance/MaintenanceView.astro`) y
  tokens de diseño en `styles/tokens.css`, siguiendo la estructura A ya
  decidida.

**Non-Goals:**
- Mecanismo de activar/desactivar modo mantenimiento (middleware, flag,
  env var). Hoy es la única página; se revisita cuando exista una segunda
  página real.
- Content Security Policy u otros headers de seguridad (el proyecto no
  tiene ninguno configurado todavía; fuera de alcance de esta vista).
- Internacionalización — el copy queda en español, igual que el boceto.

## Decisions

### 1. Tailwind v4 vía `@tailwindcss/vite`, no `@astrojs/tailwind`
`@astrojs/tailwind` es la integración legacy pensada para Tailwind v3 y no
se mantiene para v4. La ruta oficial actual es el plugin de Vite
(`@tailwindcss/vite`) agregado directo en `astro.config.mjs` bajo
`vite.plugins`. Además la paleta default de Tailwind v4 ya usa `oklch`,
que es el mismo espacio de color que usa el boceto — no hace falta
traducir colores a hex/rgb.

### 2. Componente extraído desde el inicio
Alternativa considerada: todo inline en `index.astro` (más simple, menos
archivos). Se descarta porque la estructura de carpetas ya está decidida
en `06-estructura-carpetas.md` y el equipo eligió no revisar esa decisión
salvo que el proyecto lo justifique — esta vista ya lo justifica al ser la
primera pieza de UI real del proyecto. `index.astro` queda delgado
(solo importa y renderiza `MaintenanceView`), la lógica de presentación
vive en el componente.

### 3. Countdown con offset explícito, no hora local del navegador
El boceto original hace `new Date('2026-12-01T00:00:00')` sin offset —
JS interpreta esa cadena como hora **local del dispositivo que la
ejecuta**, así que un visitante fuera de México vería una cuenta
regresiva distinta (adelantada o atrasada) a uno en Monterrey. Se fija el
offset explícito de Monterrey/Nuevo León, que desde 2022 no observa
horario de verano: `2026-12-01T00:00:00-06:00`. Así todos los visitantes
ven el mismo conteo, sin importar su zona horaria.

### 4. GSAP solo client-side, sin tocar el render de servidor
Astro con `output: 'server'` renderiza el HTML/CSS en el servidor; el
`<script>` que inicializa GSAP (marquee, entrada, flip de dígitos) corre
únicamente en el cliente, como cualquier script de Astro. No hay
hidratación de framework de por medio (no es un island de React/Vue) —
es DOM + GSAP directo, más simple que meter un framework de UI solo para
esta pantalla.

### 5. Flip de dígitos: comparación por segmento, no re-render completo
Cada segmento (días/horas/min/seg) guarda su último valor renderizado
(`data-value` o variable JS). En cada tick (cada segundo), solo se
dispara la animación GSAP en los segmentos cuyo valor cambió — evita
animar los 4 recuadros cada segundo cuando, por ejemplo, solo cambian los
segundos.

### 6. Copy "LANZA:" se mantiene igual al boceto
Se conserva el texto original en vez de suavizarlo a "LISTO:" o "META:".
Decisión del usuario: la banda inferior sigue diciendo
"LANZA: {{launchLabel}}" apuntando a `2026-12-01`.

## Risks / Trade-offs

- **Tailwind v4 + Astro 7, integración relativamente nueva** → Mitigación:
  seguir la guía oficial de Astro para Tailwind v4 (`@tailwindcss/vite`),
  verificar con `pnpm run dev` y `pnpm run build` antes de dar por
  terminado (ya es requisito de `AGENTS.md`).
- **GSAP agrega peso al bundle de una página que es 100% estática en
  contenido** → Aceptado: el usuario autorizó explícitamente la
  dependencia: el motivo es la calidad de la animación, no performance
  pura.
- **`oklch()` no soportado en navegadores muy viejos (Safari <15.4)** →
  Riesgo ya existente en el boceto original, se hereda sin cambio; no es
  parte del alcance de esta vista arreglarlo.
- **Countdown llega a cero y el proyecto sigue en desarrollo** → El
  `tick()` original ya usa `Math.max(0, diff)`, así que se congela en
  `00 00 00 00` en vez de mostrar negativos. Se conserva ese
  comportamiento; no se agrega mensaje especial de "ya lanzamos" (fuera
  de alcance).

## Migration Plan

Cambio greenfield, sin estado ni datos que migrar:
1. Instalar `tailwindcss`, `@tailwindcss/vite`, `gsap`.
2. Configurar el plugin de Tailwind en `astro.config.mjs`.
3. Crear `src/styles/tokens.css` con los tokens de diseño del boceto.
4. Crear `src/components/maintenance/MaintenanceView.astro`.
5. Reescribir `src/pages/index.astro` para renderizar el componente.
6. Verificar `pnpm run dev` (sin errores de consola) y `pnpm run build`.

Rollback: revertir el commit — no hay datos persistidos ni migraciones de
base de datos involucradas.

## Open Questions

Ninguna pendiente.

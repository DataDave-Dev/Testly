## 1. Dependencias y configuración

- [x] 1.1 Instalar `tailwindcss` y `@tailwindcss/vite`
- [x] 1.2 Instalar `gsap`
- [x] 1.3 Agregar el plugin de Tailwind a `vite.plugins` en
      `astro.config.mjs`

## 2. Tokens de diseño

- [x] 2.1 Crear `src/styles/tokens.css` con los colores oklch, fuentes
      (Archivo Black, Space Grotesk) y directiva `@import "tailwindcss"`
- [x] 2.2 Agregar el `<link>` de Google Fonts (Archivo Black + Space
      Grotesk) en el `<head>` de la página

## 3. Componente de la vista

- [x] 3.1 Crear `src/components/maintenance/MaintenanceView.astro` con el
      markup traducido del boceto (título, subtítulo, tarjetas de
      countdown, dos bandas de marquee), usando clases Tailwind en vez de
      `style=""` inline
- [x] 3.2 Quitar todo rastro del tool de diseño: `x-dc`, `support.js`,
      `data-dc-script`, clase `DCLogic`, placeholders `{{}}`
- [x] 3.3 Reescribir `src/pages/index.astro` para importar y renderizar
      `MaintenanceView`, sin el `<h1>Astro</h1>` del scaffold

## 4. Lógica del countdown

- [x] 4.1 Implementar el cálculo del tiempo restante contra
      `2026-12-01T00:00:00-06:00` (offset explícito, no hora local del
      navegador)
- [x] 4.2 Actualizar el countdown cada segundo con `setInterval`,
      limpiando el intervalo si aplica
- [x] 4.3 Clampear a `00` en los cuatro segmentos cuando la fecha ya se
      alcanzó (sin negativos)
- [x] 4.4 Formatear la etiqueta de fecha (`launchLabel` del boceto) en
      español

## 5. Animaciones GSAP

- [x] 5.1 Timeline infinito para la banda superior (scroll izquierda)
- [x] 5.2 Timeline infinito para la banda inferior (scroll derecha,
      dirección opuesta)
- [x] 5.3 Animación de entrada con stagger (título → subtítulo →
      tarjetas) al cargar la página
- [x] 5.4 Animación de flip por segmento del countdown, disparada solo
      cuando ese segmento cambia de valor respecto al tick anterior

## 6. Verificación

- [x] 6.1 `pnpm run dev` — la vista carga sin errores en consola,
      countdown y animaciones funcionan
- [x] 6.2 `pnpm run build` — build sin errores
- [x] 6.3 `pnpm run lint` — sin errores
- [x] 6.4 Confirmar en el navegador que dos zonas horarias distintas (o
      simular cambiando la zona del sistema) muestran el mismo countdown

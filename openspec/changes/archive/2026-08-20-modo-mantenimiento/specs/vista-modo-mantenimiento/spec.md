## ADDED Requirements

### Requirement: Página única del sitio
La ruta raíz (`/`) SHALL servir la vista de modo mantenimiento como único
contenido del sitio. Ninguna otra ruta pública SHALL existir en este
cambio.

#### Scenario: Visita a la raíz
- **WHEN** un visitante entra a `/`
- **THEN** recibe la vista de modo mantenimiento (título, countdown y
  bandas animadas), no el scaffold por defecto de Astro

#### Scenario: Sin rutas adicionales
- **WHEN** se inspecciona `src/pages/`
- **THEN** `index.astro` es el único archivo de página

### Requirement: Countdown a fecha fija con zona horaria explícita
El countdown SHALL calcular el tiempo restante hasta
`2026-12-01T00:00:00-06:00`, con offset de zona horaria explícito en el
código (no hora local implícita del navegador del visitante).

#### Scenario: Mismo conteo sin importar la zona horaria del visitante
- **WHEN** dos visitantes en zonas horarias distintas cargan la página en
  el mismo instante
- **THEN** ambos ven el mismo valor de días/horas/minutos/segundos
  restantes

#### Scenario: Actualización cada segundo
- **WHEN** la página lleva más de un segundo cargada
- **THEN** los segundos mostrados se actualizan cada segundo sin recargar
  la página

#### Scenario: Fecha ya alcanzada
- **WHEN** la hora actual es igual o posterior a
  `2026-12-01T00:00:00-06:00`
- **THEN** el countdown muestra `00` en los cuatro segmentos (días, horas,
  minutos, segundos), sin valores negativos

### Requirement: Sin mecanismo de activación/desactivación
La vista de modo mantenimiento SHALL renderizarse siempre, sin flag,
variable de entorno ni middleware que la active o desactive.

#### Scenario: No existe toggle
- **WHEN** se revisa el código del proyecto (middleware, variables de
  entorno, configuración)
- **THEN** no hay ningún mecanismo condicional que decida si se muestra
  la vista de mantenimiento u otro contenido

### Requirement: Animación de marquee con GSAP
Las dos bandas de texto en scroll continuo (superior e inferior) SHALL
animarse con timelines de GSAP en bucle infinito, en direcciones opuestas,
en vez de `@keyframes` CSS.

#### Scenario: Bandas en movimiento continuo
- **WHEN** la página termina de cargar
- **THEN** ambas bandas de texto se desplazan horizontalmente sin
  detenerse ni saltos visibles al reiniciar el bucle

#### Scenario: Direcciones opuestas
- **WHEN** se observa el movimiento de ambas bandas
- **THEN** una se desplaza hacia la izquierda y la otra hacia la derecha

### Requirement: Animación de entrada al cargar
Los elementos principales (título, subtítulo, tarjetas del countdown)
SHALL animarse de entrada con GSAP al cargar la página, con stagger entre
ellos.

#### Scenario: Secuencia de aparición
- **WHEN** la página carga por primera vez
- **THEN** el título aparece antes que el subtítulo, y el subtítulo antes
  que las tarjetas del countdown

### Requirement: Animación de flip por dígito
Cada segmento del countdown (días, horas, minutos, segundos) SHALL
animarse individualmente con GSAP cuando su valor cambia respecto al tick
anterior, y SHALL permanecer sin animar cuando su valor no cambia.

#### Scenario: Solo el segmento que cambia se anima
- **WHEN** pasa un segundo y solo el valor de "segundos" cambia
- **THEN** únicamente la tarjeta de segundos dispara su animación de flip;
  días, horas y minutos permanecen sin animar

### Requirement: Estilos con Tailwind CSS
El markup de la vista SHALL usar clases de utilidad de Tailwind CSS v4
para layout, tipografía y color, en vez de estilos inline como en el
boceto original.

#### Scenario: Sin estilos inline heredados del boceto
- **WHEN** se inspecciona `MaintenanceView.astro`
- **THEN** el layout, espaciado y color se expresan con clases de
  Tailwind, no con atributos `style=""` copiados del `.dc.html` original

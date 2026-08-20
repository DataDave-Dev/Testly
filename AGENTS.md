## Estilo de código

Código simple. Nada de sobreingeniería.

- Preferir la solución más directa que funcione, no la más "elegante" o abstracta.
- Sin abstracciones, capas ni configuración para casos hipotéticos futuros (YAGNI).
- Legible antes que "clever": otra persona del equipo debe entenderlo sin explicación.
- No introducir patrones de diseño, interfaces o factories si un caso concreto no lo pide ya.
- Preferir pocas líneas claras sobre muchas líneas "flexibles".

## Pruebas antes de PR

Antes de abrir una PR, verificar en local:

- `pnpm run dev` — el cambio funciona en desarrollo, sin errores en consola.
- `pnpm run build` — build sin errores.
- Tests — deben pasar (si el proyecto todavía no tiene suite de tests, no aplica).
- Lint — sin errores (si el proyecto todavía no tiene lint configurado, no aplica).

No abrir PR si alguno de estos falla.

## Resolución de problemas

Al resolver un bug o problema, seguir estos pasos en orden:

1. Identificar el problema.
2. Reproducir el error.
3. Investigar qué lo provoca.
4. Encontrar la causa raíz.
5. Diseñar una solución.
6. Implementar el cambio.
7. Probar la solución.
8. Verificar que no rompió otra cosa.
9. Documentar el cambio.
10. Desplegar y monitorear.

## Comentarios

Sin comentarios largos ni encabezados explicando qué hace cada archivo/función.

- Comentario solo si la lógica es complicada por naturaleza y el equipo la necesita para entenderla.
- Nada de comentario al inicio de cada `.ts` resumiendo el archivo.
- Código autoexplicativo (buenos nombres) en vez de comentario.

## Dependencias

No instalar paquetes ni dependencias nuevas sin autorización explícita.

- Si un problema se resuelve con código propio simple, no se agrega una librería para eso.
- Antes de agregar cualquier dependencia, preguntar y esperar aprobación.

## Development

When starting the dev server, use background mode:

```
astro dev --background
```

Manage the background server with `astro dev stop`, `astro dev status`, and `astro dev logs`.

## Documentation

Full documentation: https://docs.astro.build

Consult these guides before working on related tasks:

- [Adding pages, dynamic routes, or middleware](https://docs.astro.build/en/guides/routing/)
- [Working with Astro components](https://docs.astro.build/en/basics/astro-components/)
- [Using React, Vue, Svelte, or other framework components](https://docs.astro.build/en/guides/framework-components/)
- [Adding or managing content](https://docs.astro.build/en/guides/content-collections/)
- [Adding styles or using Tailwind](https://docs.astro.build/en/guides/styling/)
- [Supporting multiple languages](https://docs.astro.build/en/guides/internationalization/)

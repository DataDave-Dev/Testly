# Arquitectura — Testly

Fecha: 2026-08-18
Versión: 0.6

## 1. Vista general

```
Navegador
  │  (isla interactiva: editor + resultado en streaming + copiar/descargar)
  ▼
Astro SSR en Cloudflare Workers  ─────────────┐
  ├── páginas y layouts (estático/servidor)   │
  │     └── landing pública (sin sesión)      │
  └── endpoints de servidor (/api/*)          │
        ├── Turnstile + rate limit si no hay  │
        │   sesión; cuota si la hay           │
        ├── valida tamaño y framework         │
        ├── llama a la API del modelo ────────┼──▶ API del modelo (Anthropic)
        └── persiste el resultado, solo       │
            si hay sesión                     │
                                              │
Turso (libSQL) ◀──────────────────────────────┘
  ├── Better Auth (sesiones por enlace mágico, opcional)
  ├── generations (historial + contabilidad de tokens, solo con sesión)
  └── anon_quota (contador por IP, sin sesión)
```

El navegador nunca habla con la API del modelo. Toda llamada pasa por el
servidor, que es donde vive la llave. La sesión es opcional: sin ella el flujo
completo funciona igual, solo sin historial y con cuota más baja.

## 2. Stack

### 2.1 Criterios de elección

Cuatro filtros, en este orden:

1. **Que el equipo pueda avanzar.** Un stack teóricamente superior que nadie
   conoce cuesta más semanas de las que tiene un semestre.
2. **Que soporte streaming.** La respuesta tarda de 20 a 45 segundos: es un
   archivo de pruebas completo más su explicación. Sin streaming la experiencia
   se rompe. Esto descarta opciones antes que cualquier otra consideración.
3. **Que no agregue costo fijo mensual.** Ya hay un costo variable por
   generación; sumarle renta de servidores estrecha el presupuesto de la API.
4. **Que sea reemplazable.** Toda la lógica de negocio vive en `src/lib/`, sin
   importar nada del framework. Si el framework estorba, se cambia el envoltorio
   y no el producto.

El punto 4 es el seguro de todo lo demás: permite equivocarse en la elección de
framework sin perder el trabajo.

### 2.2 Framework

| Opción | A favor | En contra |
|---|---|---|
| **Astro + adapter SSR** | Preferencia del equipo. Islas: solo se envía JS donde hay interacción. Endpoints de servidor simples. Las páginas que no son la app cargan casi instantáneas | No trae auth ni capa de datos. Menos ejemplos de "app con sesión". El modelo de islas confunde al principio |
| Next.js (App Router) | El ecosistema más grande para exactamente esta forma de aplicación. Streaming, auth y rutas de servidor resueltos y documentados | Más pesado. La distinción server/client component tiene su propia curva. Acopla a un modelo de despliegue |
| SvelteKit | Menos código para lo mismo. Bundle ligero. Streaming nativo | Comunidad menor. Solo conviene si alguien del equipo ya sabe Svelte |
| Nuxt | Equivalente en Vue | Mismo caso: solo si el equipo ya es de Vue |
| React + backend Python (FastAPI o Django) | Separación clara front/back. Python es el lenguaje de más clases | Dos proyectos, dos despliegues, dos lenguajes. Para un equipo escolar es duplicar el trabajo de infraestructura |

**Elegido: Astro con adapter SSR.** Es la preferencia del equipo y encaja con la
forma real del producto: dos o tres pantallas y un endpoint que hace el trabajo
pesado. No es un producto con decenas de vistas con estado compartido, que es
donde Astro empezaría a estorbar.

**Plan B: Next.js.** Si en la Fase 1 el adapter o el streaming dan problemas que
cuesten más de unos días, se cambia. Gracias al criterio 4 el cambio toca la
carpeta `pages/` y poco más.

#### Lo que hay que verificar temprano

- `output: 'server'` y un adapter instalado. Sin eso no existen los endpoints.
- Que el endpoint pueda devolver un `ReadableStream` y que llegue al navegador
  sin buffering intermedio, **en el hosting elegido**, no solo en local. Esta
  prueba va en la Fase 1, antes de escribir interfaz.

### 2.3 Framework de las islas

Solo hay dos islas: el editor y el panel de resultado. La elección pesa poco.

| Opción | Cuándo |
|---|---|
| React | Por defecto. Más tutoriales, más gente lo conoce, funciona con casi cualquier librería |
| Preact | Igual API que React, bundle mucho menor. Si el peso importa |
| Svelte | Menos código. Si el equipo ya lo usa |
| Vanilla / Astro sin framework | Posible, pero el panel de resultado ya no es solo texto: tiene bloque de código, botón de copiar y de descargar |

**Elegido: React.** Es la opción por defecto razonable: más tutoriales, más
gente lo conoce y compatible con CodeMirror si se agrega en la Fase 4.

### 2.4 Auth y base de datos

Cambio de decisión respecto de una versión anterior de este documento: el login
dejó de ser obligatorio. La generación funciona sin cuenta, con cuota reducida
por IP; crear sesión (sin contraseña, por enlace mágico) solo sube la cuota y
habilita el historial. Eso cambia qué se necesita de la base de datos y de la
capa de auth.

#### Base de datos

| Opción | A favor | En contra |
|---|---|---|
| **Turso (libSQL)** | SQLite distribuido en el edge: misma filosofía de latencia que Cloudflare Workers. Gratis sin tarjeta: 5 GB, 500M filas leídas/mes, 10M escritas/mes — muy por encima de lo que este proyecto necesita | No trae auth ni RLS: hay que resolver aislamiento entre usuarios a mano en las consultas |
| Supabase | Postgres + auth + Row Level Security en un solo servicio | El plan gratis pausa proyectos inactivos. Menos natural con un runtime que no es Node completo |
| Neon (Postgres) | Postgres puro, buen plan gratis | Mismo problema que Supabase: no trae auth, y se pierde la ventaja de latencia del edge frente a Turso |
| PocketBase | Un binario, SQLite, auth incluida | Requiere servidor propio. No encaja con Cloudflare Workers |

**Sin connection pool.** El cliente `@libsql/client/web` (import obligatorio en
Workers; el paquete normal no funciona ahí) habla el protocolo Hrana sobre
HTTP/WebSocket, no TCP. Cada consulta es en esencia un `fetch()`, así que no
hay conexión persistente que multiplicar por instancia — el problema clásico de
serverless contra Postgres (de ahí PgBouncer, el pooler de Supabase, el de
Neon) no aplica aquí. El cliente se crea por petición o se cachea en el binding
`Env`, siguiendo el ejemplo oficial de Cloudflare para Turso.

**Elegido: Turso.** Con login opcional, la ventaja de Supabase (auth integrada)
pesa menos, y la de Turso (edge, gratis con margen amplio, encaja con
Cloudflare) pesa más.

#### Autenticación

| Opción | A favor | En contra |
|---|---|---|
| **Better Auth** | Soporta sesión por enlace mágico (sin contraseña) de fábrica. Adaptador para libSQL/Turso. Guía oficial para Astro | Librería más joven que las alternativas históricas |
| Lucia | — | **Descartada: el mantenedor la archivó en marzo de 2025.** Dejó de ser librería mantenida y quedó como guía de referencia. Cualquier tutorial que la recomiende está desactualizado |
| Auth.js / NextAuth | Maduro, muchos adaptadores | Pensado para Next.js; la integración con Astro es menos directa |
| Auth propia con sesiones en la base | Cero dependencias externas | Es el clásico "esto lo saco en un rato" que se come dos semanas y termina con un bug de seguridad |

**Elegido: Better Auth**, configurado solo con el proveedor de enlace mágico
(sin contraseña, sin OAuth). Resuelve exactamente RF-01: crear sesión con
correo, nada más.

#### Aislamiento entre usuarios, sin RLS

Turso no tiene Row Level Security. El aislamiento se hace a mano: toda consulta
a `generations` filtra por `user_id = ?` en el código del servidor, nunca se
confía en el cliente para eso. Es más frágil que RLS —un error humano en una
consulta sí puede filtrar datos de otro usuario— así que las consultas a esa
tabla deben vivir en un solo módulo (`src/lib/db/generations.ts`) y no
repetirse sueltas por el código.

### 2.5 Editor de código

| Opción | Peso | Da |
|---|---|---|
| **`textarea`** | 0 KB | Pegar y editar texto |
| CodeMirror 6 | ~150 KB | Resaltado de sintaxis, números de línea |
| Monaco (el de VS Code) | Megabytes | Un IDE completo |

**MVP: `textarea`.** El usuario pega código y lo envía; no lo escribe ahí. El
resaltado es agradable pero no cambia si el producto funciona.

**Fase 4, si sobra tiempo: CodeMirror 6.** Monaco está descartado: pesa más que
todo el resto de la aplicación junta para un caso de "pegar 100 líneas".

### 2.6 Renderizado del resultado

El modelo devuelve Markdown, que hay que convertir a HTML **en el navegador y en
streaming**. Esto descarta el Markdown integrado de Astro, que trabaja en tiempo
de compilación.

- **`marked` + `DOMPurify`** — la combinación estándar. Ligera y suficiente.
- `markdown-it` + sanitizador — equivalente, algo más configurable.

Regla no negociable: la salida del modelo pasa **siempre** por el sanitizador
antes de tocar el DOM. Nada de `set:html` ni `innerHTML` con texto crudo del
modelo (RNF-06).

**El bloque de código es ahora el elemento principal**, no un detalle del
formato. De ahí tres consecuencias:

1. **Extracción del archivo.** El prompt exige el archivo de pruebas completo en
   **un solo bloque cercado**, bajo un encabezado fijo. Se extrae con una
   expresión regular sobre el Markdown acumulado y alimenta los botones de
   copiar y descargar (RF-10, RF-11). No hace falta parsear nada más.
2. **Parpadeo durante el stream.** Un bloque cercado a medio abrir se renderiza
   roto. Se acumula el texto y se re-renderiza cada ~100 ms con un `setTimeout`,
   en lugar de por cada fragmento recibido. Con un archivo de pruebas completo
   llegando en streaming esto pasa de cosmético a necesario.
3. **Resaltado de sintaxis del resultado (opcional, Fase 4).** Si se agrega, que
   sea Shiki en el servidor o `highlight.js` con solo los lenguajes del MVP
   (Python, JavaScript/TypeScript) cargados. Importar un resaltador completo
   por estética cuesta más que todo el resto de la página.

La descarga es un `Blob` más un `<a download>`: no requiere endpoint. El nombre
del archivo sale de una tabla por framework (ver sección 5.2).

### 2.7 Estilos

| Opción | A favor | En contra |
|---|---|---|
| **Tailwind** | Velocidad de iteración. Estilos junto al marcado. Buena integración con Astro | Sin configurar tokens, el resultado se ve como todos los demás proyectos con Tailwind |
| CSS propio con custom properties | Control total. Tokens de diseño explícitos. Cero dependencias | Más lento al inicio. Requiere disciplina de organización |

**Elegido: Tailwind con tokens propios.**

La condición que hace que esta decisión funcione: **definir la paleta, la
tipografía y la escala de espaciado antes de escribir componentes**, y
configurarlas en el tema de Tailwind. Usar la paleta por defecto es lo que hace
que un proyecto con Tailwind se vea igual que todos los demás.

Concretamente, antes del primer componente de la Fase 4 debe existir un archivo
de tokens con:

- Paleta: superficie, texto, acento, y estados de advertencia / éxito / error
- **Una tipografía monoespaciada elegida a propósito.** En este producto el
  código no es un detalle: es la mitad de la pantalla, en la entrada y en la
  salida. La fuente monoespaciada por defecto del navegador es la señal más
  rápida de que a nadie le importó
- Escala de espaciado y de tamaños de texto

Sin ese archivo, Tailwind es un generador de interfaces genéricas.

### 2.8 Hosting

Aquí manda un solo requisito: **la función que hace el streaming tiene que poder
correr 60 segundos o más.** Es el filtro que descarta opciones más rápido que
cualquier comparación de precios.

El umbral sube respecto de una versión anterior de este documento: generar un
archivo de pruebas completo es una respuesta bastante más larga que explicar
código, y la latencia crece con los tokens de salida. RNF-01 pide 45 segundos en
el percentil 90, lo que significa que una de cada diez generaciones tarda más.

| Opción | A favor | A verificar |
|---|---|---|
| **Cloudflare Workers** | **Sin límite de tiempo de pared en peticiones HTTP** mientras el cliente siga conectado: el streaming largo deja de ser un riesgo. Adapter oficial de Astro (`@astrojs/cloudflare`). Plan gratis | El límite real es de **tiempo de CPU**: 10 ms en el plan gratis, hasta 5 min en el de $5/mes. Esperar al modelo es I/O, no CPU, así que no debería morder — verificar en la Fase 1 |
| Vercel | Adapter oficial de Astro. Despliegue desde git. Plan gratis | Límite de duración de función de **60 segundos** en el plan gratis. Con RNF-01 en 45 s p90, el margen es de solo 15 s, y la Fase 2 (probar `effort: high`) puede rebasarlo directamente |
| Netlify | Equivalente a Vercel, también con adapter | El mismo límite de duración que Vercel |
| Fly.io / Railway / VPS | Node completo, sin límites raros de duración ni de CPU. Los compiladores de Java/C++ vienen con la imagen si se necesita validación real (ver sección 2.11) | Hay que administrarlo. Implica costo fijo mensual |

**Elegido: Cloudflare Workers**, con el adapter `@astrojs/cloudflare`.

La razón no es el precio, es el streaming. RNF-01 acepta que el 10% de las
generaciones tarde más de 45 segundos; el techo de Vercel gratis son 60. Ese
margen es demasiado ajustado para un proyecto que además necesita medir
`effort: high` en la Fase 2, donde el modelo razona más y tarda más. En
Cloudflare ese riesgo no existe: el límite es de CPU, no de tiempo de pared, y
esperar la respuesta del modelo no consume CPU.

**Plan B: Vercel**, con el adapter `@astrojs/vercel`. Si Cloudflare da problemas
de compatibilidad con el SDK del modelo (su runtime no es Node completo) que
cuesten más de un día, se cambia. Gracias al criterio 4 de la sección 2.1 el
cambio toca el adapter y poco más.

La verificación del límite de duración sigue siendo obligatoria en la Fase 1,
sea cual sea el hosting: se despliega un endpoint que haga streaming durante 60
segundos y se confirma que llega completo al navegador **desde el dominio
desplegado**, no desde `localhost`. En local nunca falla; el límite solo aparece
en producción.

Si aun con Cloudflare el peor caso da problemas, hay dos salidas, en orden de
esfuerzo: recortar el peor caso bajando `effort` (que además abarata, ver
[03-costos.md](03-costos.md)), o pasar al plan de $5/mes de Cloudflare, que sube
a 30 segundos de CPU por defecto (configurable hasta 5 minutos). Ese plan de
$5/mes, para contexto, cuesta más en cuatro meses que toda la API del modelo
durante el semestre — ver [05-proveedor-llm.md](05-proveedor-llm.md), sección 4.3.

### 2.9 Stack recomendado — resumen

| Capa | Elección | Estado |
|---|---|---|
| Framework | Astro con adapter SSR (`@astrojs/cloudflare`) | Decidido |
| Islas | React | Decidido |
| Auth + BD | Turso (libSQL) + Better Auth (magic link, sesión opcional) | Decidido |
| Acceso a datos | Cliente `@libsql/client` directo | Decidido |
| Cliente del modelo | `@anthropic-ai/sdk`, solo en servidor | Decidido |
| Editor | `textarea` en MVP, CodeMirror 6 después | Decidido |
| Render Markdown | `marked` + `DOMPurify` | Decidido |
| Descarga | `Blob` + `<a download>`, sin endpoint | Decidido |
| Estilos | Tailwind con tokens propios, monoespaciada elegida | Decidido |
| Hosting | Cloudflare Workers, sin límite de tiempo de pared. Plan B: Vercel | Decidido |
| Validación de la salida | Aserciones por framework (texto/regex), no parseo pesado | Decidido |

Con dos tablas propias (`generations`, `anon_quota`) más las que crea Better
Auth, el cliente `@libsql/client` con SQL directo alcanza. Drizzle se
justificaría solo si el esquema crece.

### 2.10 Lo que deliberadamente no se usa

| Descartado | Por qué |
|---|---|
| Sandbox de ejecución (Docker, Firecracker, E2B, Judge0) | Decisión de producto: no se ejecuta nada. Es lo que mantiene el proyecto dentro de un semestre. Ver sección 7 |
| Parseo real de sintaxis (tree-sitter, `ast`, compiladores) en el MVP | Descartado por ahora, no para siempre. Los techos de Cloudflare gratis (10 ms CPU, 3 MB de bundle) lo hacen incómodo, y las aserciones por framework de la sección 2.11 cubren el modo de falla real con costo casi cero. Se reevalúa si la Fase 2 no alcanza el criterio de validez |
| ORM completo (Prisma, TypeORM) | Hay una tabla. El costo de configuración y de arranque supera el beneficio |
| Redis o cola de trabajos | No hay trabajo asíncrono: la llamada al modelo es síncrona con streaming. Una cola aquí solo agrega puntos de falla |
| Docker para desarrollo | La base de datos es remota. `npm run dev` basta |
| Monorepo, workspaces | Es un proyecto |
| Estado global (Redux, Zustand) | Dos islas que no comparten estado entre sí |
| Framework de componentes (shadcn, MUI) | Traen su propia estética reconocible. Con cinco o seis componentes propios se ve intencional y pesa menos |

### 2.11 Validación del archivo generado

Con Cloudflare, la validación pesada (parseo real de sintaxis con `tree-sitter`
o con un compilador) queda limitada por dos techos del plan gratis: 10 ms de CPU
y 3 MB de bundle comprimido. Ninguno de los paquetes de parseo cabe ahí con
comodidad.

Lo que sí cabe, porque cuesta microsegundos y nada de bundle, son **aserciones
por framework**: comparaciones de texto y expresiones regulares que verifican
que el archivo generado tenga la forma esperada. Por ejemplo, para pytest: que
el archivo contenga funciones `def test_` y **no** contenga `import unittest`
mezclado con fixtures de pytest (mezcla de framework, error común que no
detecta un parser de sintaxis porque es sintácticamente válido). Para Vitest:
que el archivo importe de `vitest` (`describe`, `it`, `expect`) y no de `jest`.

(Java y C/C++ quedan fuera del MVP — ver [01-prd.md](01-prd.md), sección 5 —
así que sus reglas, JUnit 5 y GoogleTest, quedan documentadas aquí solo como
referencia para cuando se agreguen.)

Esto ataca el modo de falla real documentado en
[05-proveedor-llm.md](05-proveedor-llm.md), sección 2, sin agregar
infraestructura ni depender del hosting. Si la Fase 2 muestra que no alcanza,
la escalada es tree-sitter compilado a WASM (funciona en Cloudflare, cuesta
unos 2-4 MB de bundle) o un servicio aparte con compiladores reales. Decisión
pendiente: ver la sección 8 de ese mismo documento.

### 2.12 Middleware y control de acceso

Con login opcional, alguien tiene que decidir en cada petición "¿hay sesión?" y
"¿esta ruta la necesita?". Eso vive en `src/middleware.ts`, el middleware
nativo de Astro — corre en cada petición, páginas y endpoints por igual, tanto
en Cloudflare como en cualquier otro adapter.

**Una sola responsabilidad: resolver sesión y proteger páginas que la exigen.**
Todo lo específico de una ruta se queda en esa ruta:

| Vive en el middleware | Vive en el endpoint |
|---|---|
| Leer la cookie de sesión y consultar a Better Auth una sola vez por petición | Verificar el token de Turnstile (solo aplica a `/api/generar` sin sesión) |
| Guardar el resultado en `context.locals.session` para que páginas y endpoints no repitan la consulta | Contar contra `anon_quota` o contra `generations`, según haya sesión o no |
| Redirigir `/historial` a la landing si no hay sesión | Validar `framework`, `lenguaje` y tamaño del código |
| Cabeceras de seguridad comunes (`X-Content-Type-Options`, etc.) | Cualquier lógica que dependa del cuerpo de la petición |

La razón de este corte: si el middleware empieza a saber de cuotas, de
Turnstile o de frameworks, se vuelve un segundo backend paralelo al que hay que
consultar para entender qué hace cualquier endpoint. Mantenerlo delgado es lo
que permite que `/api/generar` siga siendo legible de arriba a abajo en un solo
archivo.

**No debe hacer rate limiting por IP.** Eso ya lo resuelve la regla de
Rate Limiting de Cloudflare (sección 7), que actúa en el borde **antes** de que
la petición llegue al Worker. Duplicarlo en el middleware sería redundante y,
peor, Cloudflare Workers no comparte estado en memoria entre instancias: sin
KV o Durable Objects, un contador en el middleware ni siquiera funcionaría bien.

**Costo de CPU a vigilar.** El plan gratis de Cloudflare da 10 ms de CPU por
petición (sección 2.8). Consultar la sesión en cada archivo estático (CSS, JS,
imágenes) sería gastar ese presupuesto en nada. El middleware debe filtrar por
`context.url.pathname` y saltarse los assets antes de tocar Better Auth:

```ts
// src/middleware.ts
import { defineMiddleware } from 'astro:middleware';
import { auth } from './lib/auth';

const RUTAS_QUE_REQUIEREN_SESION = ['/historial', '/api/generaciones'];

export const onRequest = defineMiddleware(async (context, next) => {
  // No tocar Better Auth para assets estáticos: cuesta CPU gratis.
  if (/\.(css|js|svg|png|ico)$/.test(context.url.pathname)) {
    return next();
  }

  const session = await auth.api.getSession({ headers: context.request.headers });
  context.locals.session = session; // null si no hay sesión: válido, no es error

  const requiereSesion = RUTAS_QUE_REQUIEREN_SESION.some((ruta) =>
    context.url.pathname.startsWith(ruta),
  );
  if (requiereSesion && session === null) {
    return context.redirect('/'); // a la landing, no a un formulario de login aparte
  }

  return next();
});
```

**Falla cerrado, no abierto.** Si Better Auth no responde (Turso caído, error de
red), `getSession` debe tratarse como "sin sesión", nunca como "autorizado".
El código de arriba ya lo hace por default: cualquier resultado que no sea una
sesión válida cae al mismo camino que "no hay sesión".

Sobre pruebas del propio proyecto: pruebas unitarias sobre `src/lib/` desde la
Fase 1 (cuota, validación de tamaño, construcción del prompt, extracción del
bloque de código). Pruebas de extremo a extremo solo sobre el flujo crítico
—pegar código, recibir pruebas, descargarlas, verlas en el historial— y hasta la
Fase 4, cuando la interfaz ya no cambia todos los días.

Que un generador de pruebas unitarias no tenga pruebas unitarias sería
señalable en la presentación. Vale la pena tenerlo presente.

### 2.13 Eliminación de cuenta (soft delete)

RF-23: el usuario puede eliminar su cuenta. Por decisión de producto **no es
un borrado físico**: sin política de retención general (el código pegado se
conserva indefinidamente mientras la cuenta exista), la única salida que tiene
el usuario para dejar de estar en la base es eliminar la cuenta, y esa acción
tiene que ser reversible por el equipo ante un error o un abuso del botón.

Better Auth expone el hook `user.deleteUser.beforeDelete`, que corre antes de
borrar físicamente y puede interrumpir el borrado lanzando un `APIError`. Se
usa así:

```ts
// src/lib/auth.ts
export const auth = betterAuth({
  // ...
  user: {
    deleteUser: {
      enabled: true,
      beforeDelete: async (user) => {
        await marcarUsuarioEliminado(user.id); // deleted_at = now(), anonimiza email/nombre
        throw new APIError('BAD_REQUEST', { message: 'soft delete aplicado' });
        // el throw evita que Better Auth ejecute el DELETE físico
      },
    },
  },
});
```

- **Qué se anonimiza:** email y nombre del `user` (para que el usuario no
  pueda volver a iniciar sesión con esa cuenta ni reclamar el correo). El
  `user.id` no cambia, así que las referencias en `generations.user_id` siguen
  siendo válidas.
- **Qué NO se toca:** las filas de `generations` de ese usuario. Se quedan
  como están, ahora huérfanas de una cuenta anonimizada — es la misma política
  de retención de siempre, solo que el dueño ya no puede iniciar sesión para
  verlas.
- **Sesión bloqueada después del soft delete:** el middleware de la sección
  2.12 ya trata "sin sesión válida" como el camino por defecto (falla
  cerrado). Basta con que `deleted_at` no sea null invalide la sesión en el
  siguiente `getSession`; Better Auth ya limpia las sesiones activas al
  eliminar el usuario (ver notas de la versión de Better Auth en uso).
- **Verificar en la Fase 3** (cuando se implementa Auth): que la versión de
  Better Auth instalada soporta `beforeDelete` tal como está documentado
  arriba, y si ya trae una opción de soft delete nativa (propuesta en su
  repositorio, sin confirmar estabilidad a la fecha de este documento) que
  ahorre escribir `marcarUsuarioEliminado` a mano.

Ver [08-limites.md](08-limites.md) para la decisión de producto detrás de esto.

## 3. Estructura de carpetas (decidida: opción A, ver [06-estructura-carpetas.md](06-estructura-carpetas.md))

```
src/
├── middleware.ts                # resuelve sesión, protege páginas (sección 2.12)
├── pages/
│   ├── index.astro              # landing: qué hace, teoría, ejemplos (RF-21)
│   ├── app.astro                # pantalla principal, sin requerir sesión
│   ├── historial.astro          # requiere sesión; sin ella, redirige a crearla
│   └── api/
│       ├── generar.ts           # POST: recibe código, responde en streaming
│       ├── auth/[...all].ts     # rutas de Better Auth (magic link, sesión)
│       └── generaciones/[id].ts # GET/DELETE: una generación del historial
├── components/
│   ├── editor/
│   │   ├── CodeInput.tsx        # isla: textarea + selectores
│   │   ├── LanguagePicker.tsx
│   │   └── FrameworkPicker.tsx  # opciones dependientes del lenguaje
│   └── result/
│       ├── ResultStream.tsx     # isla: consume el streaming y renderiza
│       ├── TestFileBlock.tsx    # bloque de código + copiar + descargar
│       └── NotRunWarning.tsx    # aviso de que no se ejecutaron (RF-12)
├── lib/
│   ├── llm/
│   │   ├── client.ts            # cliente del modelo
│   │   ├── prompt.ts            # prompt de sistema (estable, cacheable)
│   │   └── generar.ts           # orquesta la llamada
│   ├── frameworks.ts            # tabla lenguaje → frameworks, nombre de archivo
│   ├── extraerTest.ts           # saca el bloque cercado del markdown
│   ├── db/
│   │   ├── generations.ts       # única puerta de entrada a la tabla; filtra por user_id
│   │   └── anonQuota.ts         # cuota por IP para el flujo sin sesión
│   └── auth.ts                  # instancia de Better Auth
└── styles/
```

`frameworks.ts` y `extraerTest.ts` son los dos módulos nuevos respecto de la
versión anterior, y los dos son lógica pura: se prueban sin levantar nada.

## 4. Flujo de una generación

1. La isla del editor hace `POST /api/generar` con
   `{ codigo, lenguaje, framework, turnstileToken }`. `turnstileToken` solo
   viaja si no hay sesión (RF-22).
2. El endpoint valida, en este orden:
   - si **no** hay sesión: verifica `turnstileToken` contra Cloudflare (si no:
     403) y cuenta contra `anon_quota` por IP (si agotada: 429)
   - si **hay** sesión: cuenta contra `generations` del usuario (si agotada: 429)
   - `framework` pertenece a `lenguaje` según `frameworks.ts` (si no: 400)
   - tamaño del código dentro del límite (si no: 413 con mensaje claro)
3. Llama al modelo en modo streaming.
4. Reenvía el stream al navegador conforme llega.
5. El navegador acumula, re-renderiza cada ~100 ms y, cuando cierra el bloque
   cercado, habilita copiar y descargar.
6. Al terminar: **con sesión**, el servidor guarda en `generations` el código,
   el resultado, el framework, el modelo usado y los tokens consumidos.
   **Sin sesión**, no se guarda la generación (RF-13); solo se incrementa el
   contador de `anon_quota` y se suman los tokens al registro de auditoría de
   gasto (RNF-08).
7. Si la llamada al modelo falla, el intento **no** descuenta cuota (RF-19).

La validación del framework contra el lenguaje va en el servidor aunque el
selector ya la haga en el cliente: el endpoint es público —más aún ahora que
no requiere sesión— y un `framework` arbitrario termina concatenado en el
prompt.

**La regla de Rate Limiting de Cloudflare** (sección 7) actúa antes de que la
petición llegue a este endpoint: es la primera capa, ciega a si hay sesión o
no. Turnstile y `anon_quota` son la segunda y tercera capa, específicas del
flujo sin cuenta.

## 5. Diseño del prompt

El prompt de sistema es el activo pedagógico del proyecto. Es lo que separa
"otro wrapper de IA" de una herramienta de estudio.

### 5.1 Puntos de diseño

- **Estable entre peticiones.** El texto no cambia por usuario ni lleva fecha ni
  identificadores. Así se puede cachear y se paga a una décima parte. Ojo con el
  mínimo cacheable, que depende del modelo (Opus 5: 512 tokens; Sonnet 5: 1,024;
  Haiku 4.5: 4,096) y que, por debajo del umbral, falla **en silencio**. Ver
  [03-costos.md](03-costos.md), sección 6.4. El
  lenguaje y el framework van en el mensaje del usuario, no en el sistema.
- **El código va delimitado y marcado como datos.** Un estudiante puede pegar
  código que contenga comentarios con instrucciones. El prompt debe indicar
  explícitamente que el contenido del bloque es material a analizar, nunca
  instrucciones a obedecer.
- **Estructura de salida fija**, en Markdown, para poder renderizar en streaming
  sin esperar a que cierre un JSON:

  ```
  ## Qué hace este código
  (dos o tres líneas; no es el producto, es contexto)

  ## Casos que vale la pena probar
  (lista: para cada caso, qué entrada, qué se espera y POR QUÉ ese caso)

  ## Archivo de pruebas
  (UN SOLO bloque cercado, el archivo completo y ejecutable)

  ## Cómo correrlo
  (el comando, en dos líneas)
  ```

  El orden importa: **el razonamiento va antes del código**. Si el archivo
  aparece primero, el estudiante lo copia y no lee lo demás, que es justamente
  lo que el producto quiere evitar. Además, en streaming, el usuario ve
  contenido útil mientras se genera la parte larga.

- **Un solo bloque cercado en toda la respuesta.** La extracción para el botón
  de descarga depende de esto. El prompt lo pide de forma explícita y prohíbe
  bloques cercados en las demás secciones.
- **Explicar el criterio, no etiquetar.** "Prueba el caso borde" no enseña nada.
  "Con una lista vacía el `for` no entra nunca y la función devuelve el
  acumulador inicial, que puede no ser lo que el autor espera" sí.
- **Cobertura obligatoria de tres familias de caso**: camino feliz, valores
  límite (vacío, cero, negativo, uno solo, el máximo) y entradas inválidas o de
  error. Es el criterio que el estudiante debe interiorizar y por eso está fijo
  en el prompt.
- **Prohibido inventar comportamiento.** Si el código depende de algo que no
  está a la vista, se afirma sobre lo que sí está. Nada de aserciones inventadas
  sobre funciones que no se enviaron.
- **Prohibidas las pruebas tautológicas.** `assert True`, aserciones que repiten
  la implementación, mocks que devuelven exactamente lo que después se afirma.
  Se prohíbe de forma explícita porque es la falla más común y la más difícil de
  notar: se ve como una prueba y no prueba nada.
- **Permitir decir que no aplica.** Si lo pegado es un script sin funciones, un
  `main` que solo imprime, o código que no se puede probar sin ejecutar efectos
  externos, el modelo lo dice y explica qué habría que cambiar para hacerlo
  probable (RF-09). Sin este permiso explícito el modelo genera pruebas
  artificiales con tal de entregar algo.
- **Nada de efectos secundarios en el código generado.** Las pruebas no deben
  tocar red, archivos, procesos ni variables de entorno. Es una regla
  pedagógica, y además de seguridad: ese archivo lo va a correr un estudiante en
  su máquina. Ver sección 7.

### 5.2 Tabla de frameworks

Vive en `src/lib/frameworks.ts` y alimenta el selector, la validación del
endpoint y el nombre del archivo descargado.

| Lenguaje | Frameworks | Archivo por defecto |
|---|---|---|
| Python | pytest (def.), unittest | `test_<nombre>.py` |
| JavaScript / TypeScript | Vitest (def.), Jest | `<nombre>.test.ts` |

`<nombre>` sale de la función o clase principal detectada por el modelo; si no
hay una clara, se usa un genérico.

Java (JUnit 5) y C/C++ (GoogleTest, Catch2) quedan fuera del MVP (ver
[01-prd.md](01-prd.md), sección 5); agregarlos es sumar dos filas a esta tabla
y sus reglas de validación (sección 2.11).

### 5.3 Boceto de la petición

```ts
// src/lib/llm/generar.ts
const stream = client.messages.stream({
  model: import.meta.env.LLM_MODEL,          // 'claude-sonnet-5', decidido
  max_tokens: 16000,                          // holgado: si se corta, el bloque
                                              // cercado nunca cierra y la
                                              // descarga sale truncada
  thinking: { type: 'adaptive' },             // no existe en Haiku 4.5
  output_config: { effort: 'medium' },        // el default es 'high': fijarlo
  system: [{
    type: 'text',
    text: PROMPT_SISTEMA,                     // estable → cacheable
    cache_control: { type: 'ephemeral' },
  }],
  messages: [{
    role: 'user',
    content:
      `Lenguaje: ${lenguaje}\n` +
      `Framework: ${framework}\n\n` +
      `Código a probar:\n\`\`\`\n${codigo}\n\`\`\``,
  }],
});
```

El nivel de `effort` es la palanca de costo y latencia más directa. Acepta
`low`, `medium`, `high`, `xhigh` y `max`, y **su valor por defecto es `high`**:
si no se fija, se paga el nivel alto sin haberlo decidido. Conviene probar
`low` y `medium` sobre el conjunto de evaluación y quedarse con el más bajo que
mantenga la validez.

Advertencia para este producto: generar código correcto es más sensible al
`effort` que explicar código, porque el razonamiento es donde el modelo verifica
que las aserciones correspondan al código real. Es plausible que `low` produzca
pruebas que no compilan. Se mide en la Fase 2.

**Dos parámetros de este boceto no existen en Haiku 4.5**: `output_config.effort`
devuelve error y el razonamiento adaptativo no está disponible. Si la Fase 2
elige Haiku por costo, el cliente necesita una rama sin esos campos y la palanca
de la sección 6.2 de costos desaparece. Es un requisito de diseño, no un
detalle: conviene que `generar.ts` construya la petición condicionalmente desde
el principio en vez de asumir que los tres modelos aceptan lo mismo.

Si más adelante se necesitan los casos por separado de forma programática (para
estadísticas de qué tipos de caso se generan, por ejemplo), se cambia a salida
estructurada con esquema JSON. Para el MVP, Markdown con encabezados fijos
alcanza y además permite el streaming legible.

## 6. Modelo de datos

Better Auth administra sus propias tablas (`user`, `session`, `verification`)
en la misma base de Turso, incluida la columna `deleted_at` en `user` que usa
el soft delete de la sección 2.13. `generations.user_id` es **nullable**: una
generación anónima no tiene usuario.

```sql
create table generations (
  id            text primary key,       -- uuid generado en la aplicación
  user_id       text references user(id) on delete cascade,  -- null si es anónima
  language      text not null,
  framework     text not null,
  code          text not null,          -- lo que pegó el estudiante
  result        text not null,          -- markdown completo devuelto por el modelo
  model         text not null,          -- qué modelo lo generó
  input_tokens  integer not null,
  output_tokens integer not null,
  created_at    text not null           -- ISO 8601; libSQL no tiene timestamptz
);

create index generations_user_created_idx on generations (user_id, created_at desc);
```

**Una generación anónima no se guarda ni siquiera temporalmente.** RF-13 lo
pide así: sin sesión, la fila ni se inserta. La tabla `generations` solo
contiene historial de usuarios con cuenta. Lo que sí hay que contar, para la
cuota de los tres gratis al día, es la **cuenta de peticiones por IP**, que no
vive en `generations` porque no está atada a un usuario:

```sql
create table anon_quota (
  ip            text not null,
  day           text not null,          -- fecha en formato YYYY-MM-DD, UTC
  count         integer not null default 0,
  primary key (ip, day)
);
```

Es la única tabla de contador del proyecto, y existe porque no hay usuario al
que atarle el conteo. Con sesión, el consumo del periodo se deriva contando
filas de `generations` directamente:

```sql
select count(*) from generations
where user_id = ? and created_at > datetime('now', '-30 days');
```

**El archivo de pruebas no se guarda en su propia columna.** Está dentro de
`result` y se extrae con la misma función que usa el botón de descarga
(`extraerTest.ts`). Guardarlo aparte sería duplicar el mismo texto en dos
columnas que pueden desincronizarse.

`input_tokens` y `output_tokens` se guardan porque sin ellos no se puede auditar
el gasto real (RNF-08), **incluidas las generaciones anónimas**: aunque no
tengan fila en `generations`, sus tokens hay que sumarlos a algún lado (un
contador simple, o un log) para que el costo real sea auditable de punta a
punta. El costo en dinero **no** se guarda: los precios cambian y un número
calculado al vuelo desde los tokens siempre es correcto.

## 7. Seguridad

| Riesgo | Mitigación |
|---|---|
| Fuga de la llave de la API | La llave solo existe como variable de entorno del servidor. Ninguna referencia en código de cliente |
| Ejecución de código malicioso en el servidor | No aplica: ni el código recibido ni las pruebas generadas se ejecutan |
| **El estudiante corre en su máquina un archivo generado con efectos secundarios** | El prompt prohíbe red, archivos, procesos y variables de entorno en el código generado. El aviso de RF-12 le dice que lo revise antes de correrlo. Es una mitigación parcial y hay que asumirlo como tal |
| Inyección de prompt vía el código pegado | El código va en bloque delimitado; el prompt de sistema declara que ese contenido son datos, no instrucciones. Refuerza la regla anterior: aunque la inyección funcione, el archivo generado sigue siendo revisado por el usuario antes de ejecutarse |
| `framework` arbitrario concatenado al prompt | Se valida contra `frameworks.ts` en el servidor; solo se aceptan valores de la lista |
| XSS al renderizar la respuesta | La salida del modelo se renderiza como Markdown con sanitizador. Nunca `innerHTML` ni `set:html` directo. Aplica también al bloque de código: se inserta como texto, no como HTML |
| **Abuso del flujo sin cuenta** (scripts que generan sin límite) | Regla de Rate Limiting de Cloudflare por IP en `/api/generar` (una sola regla disponible en el plan gratis) + Cloudflare Turnstile antes de generar sin sesión (RF-22) + tope de 3/día por IP en `anon_quota`. Tres capas porque ninguna sola alcanza: el rate limit de Cloudflare frena ráfagas, Turnstile frena scripts, la tabla frena el conteo real |
| Un usuario lee el código de otro | Turso no tiene RLS: el aislamiento se hace a mano, filtrando `user_id` en cada consulta desde un único módulo (`src/lib/db/generations.ts`). Ver sección 2.4 |
| Secretos pegados por el usuario | El editor advierte que no se peguen credenciales. Con sesión, el código queda en el historial del usuario y es visible para él. Sin sesión, no se guarda nada |

La tercera fila es el riesgo nuevo que introduce este producto y no existía en
la versión anterior de la idea: **antes la salida era prosa para leer, ahora es
código para ejecutar.** No se resuelve del todo sin ejecutar las pruebas en un
sandbox, que está fuera del alcance. La postura es prohibirlo en el prompt y
decírselo al usuario, no fingir que el riesgo desapareció.

## 8. Variables de entorno

```
LLM_API_KEY=            # llave de la API del modelo (solo servidor)
LLM_MODEL=claude-sonnet-5       # decidido; ver 03-costos.md sección 6.1
LLM_EFFORT=medium               # fijarlo: el default de la API es 'high'
TURSO_DATABASE_URL=
TURSO_AUTH_TOKEN=       # solo servidor
BETTER_AUTH_SECRET=     # solo servidor
RESEND_API_KEY=         # o el proveedor de correo elegido, para el magic link
TURNSTILE_SECRET_KEY=   # solo servidor; la site key sí es pública
MAX_CODE_CHARS=12000            # ~3,000 tokens de código como máximo
LLM_MAX_OUTPUT_TOKENS=16000     # ver 08-limites.md; si se corta, la descarga sale truncada
QUOTA_ANONIMA_DIARIA=3          # decidido; sin sesión, por IP
QUOTA_PER_PERIOD=10             # decidido; con sesión, por 30 días
```

El servidor debe verificar al arrancar que las variables requeridas existen y
fallar de inmediato si falta alguna. Un despliegue sin llave que falla en la
primera petición del usuario es peor que uno que no arranca.

`LLM_MODEL` y las cuotas quedan como variables de entorno con su valor final ya
puesto, no como placeholders: son la configuración decidida en
[03-costos.md](03-costos.md), sección 9. Siguen siendo variables (no
constantes en código) porque `LLM_EFFORT` es lo único que la Fase 2 todavía
puede mover, y porque si el hosting corta por duración, bajarlo es la primera
salida.

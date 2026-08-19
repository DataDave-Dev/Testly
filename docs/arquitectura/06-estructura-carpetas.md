# Estructura de carpetas — Testly

Fecha: 2026-08-18
Versión: 0.2 — **Decidido: opción A (por tipo, convención de Astro)**

Cinco formas de organizar `src/`, con el mismo inventario real de archivos que
ya salió en [02-arquitectura.md](02-arquitectura.md): `middleware.ts`, 3
páginas, 3 rutas de API, 6 componentes (editor + resultado), y en `lib/` el
cliente del modelo, el prompt, `frameworks.ts`, `extraerTest.ts`, dos módulos
de base de datos y la instancia de auth.

El equipo eligió **A**. No hay que crear ni mover nada: es exactamente la
estructura que ya está bocetada en [02-arquitectura.md](02-arquitectura.md),
sección 3 — esta elección solo la confirma como definitiva en vez de
"propuesta". Las opciones B-E quedan documentadas abajo como referencia, por si
el proyecto crece y hace falta revisar la decisión.

## A) Por tipo de archivo (convención de Astro)

```
src/
├── middleware.ts
├── pages/
│   ├── index.astro              # landing
│   ├── app.astro                # pantalla principal
│   ├── historial.astro
│   └── api/
│       ├── generar.ts
│       ├── auth/[...all].ts
│       └── generaciones/[id].ts
├── components/
│   ├── editor/
│   │   ├── CodeInput.tsx
│   │   ├── LanguagePicker.tsx
│   │   └── FrameworkPicker.tsx
│   └── result/
│       ├── ResultStream.tsx
│       ├── TestFileBlock.tsx
│       └── NotRunWarning.tsx
├── lib/
│   ├── llm/
│   │   ├── client.ts
│   │   ├── prompt.ts
│   │   └── generar.ts
│   ├── frameworks.ts
│   ├── extraerTest.ts
│   ├── db/
│   │   ├── generations.ts
│   │   └── anonQuota.ts
│   └── auth.ts
└── styles/
    └── tokens.css
```

Cada carpeta responde "¿qué tipo de cosa es esto?": páginas aquí, componentes
allá, lógica de negocio en `lib/`. Es la convención que usa la mayoría de
proyectos Astro y la que ya está bocetada en el resto de la documentación.

**A favor:** cualquier tutorial de Astro encaja sin traducir nada; el equipo no
tiene que inventar una convención propia. **En contra:** para entender "todo lo
que toca el historial" hay que saltar entre `pages/historial.astro`,
`api/generaciones/[id].ts`, `components/result/` y `lib/db/generations.ts` —
cuatro carpetas para una sola funcionalidad.

## B) Por feature (vertical slice)

```
src/
├── middleware.ts
├── pages/                       # solo rutas, delgadas
│   ├── index.astro
│   ├── app.astro
│   ├── historial.astro
│   └── api/
│       ├── generar.ts           # importa de features/generar/
│       ├── auth/[...all].ts
│       └── generaciones/[id].ts
├── features/
│   ├── generar/
│   │   ├── CodeInput.tsx
│   │   ├── LanguagePicker.tsx
│   │   ├── FrameworkPicker.tsx
│   │   ├── ResultStream.tsx
│   │   ├── TestFileBlock.tsx
│   │   ├── NotRunWarning.tsx
│   │   ├── frameworks.ts
│   │   ├── extraerTest.ts
│   │   └── llm/
│   │       ├── client.ts
│   │       ├── prompt.ts
│   │       └── generar.ts
│   ├── historial/
│   │   └── generations.ts
│   └── auth/
│       ├── auth.ts
│       └── anonQuota.ts
└── styles/
    └── tokens.css
```

Cada carpeta responde "¿a qué funcionalidad pertenece esto?". Coincide con la
regla propia del equipo de organizar por feature y no por tipo de archivo.

**A favor:** todo lo de "generar" —que es el 80% del proyecto— vive junto,
componentes y lógica en la misma carpeta. **En contra:** con solo dos features
reales (generar, historial) y un `auth` que en realidad cruza a las otras dos,
la ganancia es menor que en un proyecto con diez features genuinamente
separadas. El beneficio de este patrón crece con el número de features; aquí
casi no hay a quién repartir.

## C) Por capa (arquitectura limpia)

```
src/
├── middleware.ts
├── pages/                       # presentación
├── application/                 # casos de uso, orquestan
│   ├── generarPruebas.ts        # llama llm + valida + persiste
│   └── gestionarHistorial.ts
├── domain/                      # lógica pura, sin dependencias externas
│   ├── frameworks.ts
│   ├── extraerTest.ts
│   └── prompt.ts
├── infrastructure/               # adaptadores a servicios externos
│   ├── llm/client.ts
│   ├── db/turso.ts
│   ├── db/generations.ts
│   ├── db/anonQuota.ts
│   └── auth/betterAuth.ts
├── components/
│   ├── editor/...
│   └── result/...
└── styles/
```

Cada carpeta responde "¿en qué capa de la arquitectura vive esto?": qué es
regla de negocio pura, qué orquesta, qué habla con el mundo exterior.

**A favor:** separación estricta; `domain/` se prueba sin mocks porque no
depende de nada externo. Escala bien si el proyecto crece mucho. **En contra:**
es la opción con más indirection de las cinco — cuatro carpetas para llegar al
código que hace el trabajo, en un proyecto con dos casos de uso reales. El
propio proyecto ya evitó este tipo de sobre-ingeniería en otras decisiones (sin
ORM, sin cola de trabajos, sin RLS porque no hacía falta la complejidad
extra) — esta estructura empuja en la dirección contraria a esas decisiones.

## D) Colocación por ruta (estilo Next App Router)

```
src/pages/
├── index.astro
├── app.astro
├── app.components/
│   ├── CodeInput.tsx
│   ├── LanguagePicker.tsx
│   ├── FrameworkPicker.tsx
│   ├── ResultStream.tsx
│   ├── TestFileBlock.tsx
│   └── NotRunWarning.tsx
├── historial.astro
├── historial.components/
│   └── ListaGeneraciones.tsx
└── api/
    ├── generar.ts
    ├── auth/[...all].ts
    └── generaciones/[id].ts

src/lib/                          # lo que no es UI sigue centralizado
├── llm/...
├── frameworks.ts
├── extraerTest.ts
├── db/...
└── auth.ts
```

Cada componente vive junto a la página que lo usa, con una convención de
nombre inventada (`*.components/`) porque Astro no tiene el equivalente
nativo de las carpetas de ruta de Next.

**A favor:** al abrir `app.astro` sus componentes están a un clic, cero
brincos para tocar la UI de una pantalla. **En contra:** Astro no tiene
convención fuerte para esto —hay que inventarla y que el equipo la respete a
mano—, y con solo dos pantallas con componentes propios (`app`, `historial`)
el beneficio es marginal frente al costo de la convención rara.

## E) Plano mínimo (YAGNI)

```
src/
├── middleware.ts
├── pages/
│   ├── index.astro
│   ├── app.astro
│   ├── historial.astro
│   └── api/
│       ├── generar.ts
│       ├── auth/[...all].ts
│       └── generaciones/[id].ts
├── components/                   # sin subcarpetas
│   ├── CodeInput.tsx
│   ├── LanguagePicker.tsx
│   ├── FrameworkPicker.tsx
│   ├── ResultStream.tsx
│   ├── TestFileBlock.tsx
│   └── NotRunWarning.tsx
├── lib/                          # sin subcarpetas
│   ├── llmClient.ts
│   ├── llmPrompt.ts
│   ├── generar.ts
│   ├── frameworks.ts
│   ├── extraerTest.ts
│   ├── dbGenerations.ts
│   ├── dbAnonQuota.ts
│   └── auth.ts
└── styles/
    └── tokens.css
```

Ninguna subdivisión más allá de "componente" o "lógica". El nombre del archivo
hace el trabajo que en otras opciones hace la carpeta.

**A favor:** 6-8 archivos por carpeta se ven completos en un `ls`, sin abrir
nada. Cero decisión de "¿esto en qué subcarpeta va?" porque solo hay un lugar
posible. **En contra:** si el alcance creciera más allá del MVP documentado
(cuentas de profesor, grupos — ya descartados explícitamente) esto se vuelve
difícil de escanear. Con el alcance actual del proyecto no debería pasar de
ahí.

## Comparación rápida

| | Archivos para entender "historial" | Carpetas nuevas a mantener | Convención que hay que inventar |
|---|---|---|---|
| A. Por tipo | 4 | 0 (ya documentada) | Ninguna |
| B. Por feature | 1 | 3 (`features/*`) | Ninguna, es estándar |
| C. Por capa | 4-5 | 4 (`application/`, `domain/`, `infrastructure/`) | Ninguna, es estándar |
| D. Por ruta | 2 | N por página | Sí, `*.components/` es invención propia |
| E. Plano | 3 (mismo número de archivos, sin carpetas) | 0 | Ninguna |

## Mi recomendación, si sirve de algo

**A o E.** El proyecto tiene dos pantallas reales, un endpoint pesado y un
equipo que ya viene de Astro por preferencia — la convención del framework
(A) es la que menos fricción agrega, y ya está semi-escrita en el resto de los
documentos. E es válida igual y quizás más honesta con el tamaño real del
proyecto: con 6-9 archivos por carpeta, las subcarpetas de A (`llm/`, `db/`,
`editor/`, `result/`) casi no ganan nada todavía.

B es la más alineada con la regla propia del equipo, pero el proyecto no tiene
suficientes features genuinamente separadas para que se note la ventaja — dos
features y un `auth` transversal no es el caso de uso donde vertical slice
brilla.

C es la que yo evitaría para este proyecto específico: el resto de las
decisiones documentadas apuntan a menos indirection, no a más, y esta va en
sentido contrario.

D depende de qué tan cómodo esté el equipo inventando y respetando una
convención que Astro no impone solo.

# Documentación del proyecto — Testly

Nombre: **Testly**.
Proyecto: plataforma web que genera pruebas unitarias a partir del código del
estudiante y le explica el criterio de cada caso, para que aprenda a escribirlas
por su cuenta.
Contexto: proyecto escolar (FIME).
Última actualización: 2026-08-19

## Índice

| Documento | Contenido |
|---|---|
| [00-idea.md](producto/00-idea.md) | Idea original y decisiones tomadas durante el mapeo |
| [01-prd.md](producto/01-prd.md) | Objetivos, justificación, alcance y requisitos |
| [02-arquitectura.md](arquitectura/02-arquitectura.md) | Stack, flujo, prompt, modelo de datos, seguridad |
| [03-costos.md](costos/03-costos.md) | Costo por generación y proyección mensual |
| [04-plan.md](gestion/04-plan.md) | Fases, entregables y riesgos |
| [05-proveedor-llm.md](costos/05-proveedor-llm.md) | Opciones gratuitas y de pago para el modelo, con datos verificados |
| [06-estructura-carpetas.md](arquitectura/06-estructura-carpetas.md) | Cinco formas de organizar `src/`, comparadas |
| [07-metodologia.md](gestion/07-metodologia.md) | Metodología de desarrollo (Scrum), equipo con matrícula, roles, sprints, ceremonias |
| [08-limites.md](gestion/08-limites.md) | Fecha límite, retención y borrado de datos, acceso, tope de salida del modelo, tabla de límites técnicos |
| [09-carta-patrocinadora.md](gestion/09-carta-patrocinadora.md) | Carta patrocinadora (project charter): beneficio, entregables, medidas de éxito, fechas y roles Scrum |

## Estructura de carpetas

Agrupado por tema, no por fecha — los números de archivo se conservan tal
cual (marcan el orden narrativo original, no la carpeta) para no romper las
referencias que ya existen en el historial de versiones.

```
docs/
├── README.md              — este índice
├── producto/               — idea y PRD
├── arquitectura/            — arquitectura y estructura de carpetas del código
├── costos/                  — costos y elección de proveedor del modelo
├── gestion/                  — plan, metodología, límites, carta patrocinadora, acta constitutiva
├── sprints/                  — bitácora por sprint
└── apoyo/                    — material de referencia (plantillas, presentaciones), no forma parte del proyecto
```

Documentación nueva entra a la carpeta que le corresponda por tema; si no
encaja en ninguna, se discute antes de crear una carpeta nueva.

## Historial de versiones

| Versión | Fecha | Cambio |
|---|---|---|
| 0.1 | 2026-08-17 | Producto planteado como analizador: explicación, errores y estilo |
| 0.2 | 2026-08-18 | **Cambio de producto.** Ahora genera pruebas unitarias y explica el criterio de cada caso. Cambian el prompt, el modelo de datos, los criterios de éxito y el costo (aproximadamente el doble) |
| 0.3 | 2026-08-18 | Precios y modelos verificados contra la documentación oficial. Sonnet 5 quedó en $2/$10 de forma permanente; se corrigen los mínimos de caché por modelo y se documenta que Haiku 4.5 no acepta `effort`. Recomendación de partida: Sonnet 5 |
| 0.4 | 2026-08-18 | **Pivotes de arquitectura y cierre de decisiones económicas.** Hosting: Cloudflare Workers en vez de Vercel (sin límite de tiempo de pared para el streaming). Base de datos: Turso en vez de Supabase, con Better Auth (Lucia está descontinuada). Login pasa de obligatorio a **opcional**: generación libre con cuota por IP, sesión solo para historial y cuota mayor. Se agregan middleware (control de acceso) y validación por aserciones de framework (RF-20). Se cierran las decisiones pendientes: el equipo paga la API ($20 USD), modelo Sonnet 5, cuota de 10/periodo con sesión y 3/día sin sesión. Con eso, [05-proveedor-llm.md](costos/05-proveedor-llm.md) queda archivado como referencia, no como plan activo |
| 0.5 | 2026-08-18 | Se agrega [07-metodologia.md](gestion/07-metodologia.md): metodología Kanban sobre las fases de 04-plan.md, equipo con matrícula, reparto sugerido por área técnica |
| 0.6 | 2026-08-18 | **Cambio de metodología a Scrum** en 07-metodologia.md (decisión del equipo, reemplaza la propuesta de Kanban). **Reducción de alcance de lenguajes**: el MVP cubre Python y JavaScript/TypeScript; Java y C/C++ quedan fuera, documentados como trabajo futuro en 00-idea.md, 01-prd.md (secciones 2 y 5), 02-arquitectura.md (sección 5.2), 03-costos.md, 04-plan.md (Fase 0 y tabla de riesgos) y 05-proveedor-llm.md (archivado, ajustado solo donde la conclusión cambiaba) |
| 0.7 | 2026-08-18 | Se agrega [08-limites.md](gestion/08-limites.md). **Fecha límite fijada: 2026-12-01** (~7 sprints disponibles, riesgo de calendario documentado en 04-plan.md y 07-metodologia.md). **RF-23 nuevo**: eliminación de cuenta por soft delete, detalle de implementación con el hook `beforeDelete` de Better Auth en 02-arquitectura.md sección 2.13. Acceso abierto a cualquiera confirmado. Tope de salida del modelo formalizado (`LLM_MAX_OUTPUT_TOKENS=16000`) |
| 0.8 | 2026-08-19 | Se agrega [09-carta-patrocinadora.md](gestion/09-carta-patrocinadora.md): carta sponsor original adaptada al proyecto actual, roles Scrum sin persona asignada dejado pendiente a propósito |
| 0.9 | 2026-08-19 | **Nombre del proyecto fijado: Testly.** Se propaga a todos los documentos que se referían a él sin nombre |
| 1.0 | 2026-08-19 | Esta carpeta pasa a versionarse en el repositorio: se retira la nota que la excluía por `.gitignore` |
| 1.1 | 2026-08-19 | **Reorganización por carpetas temáticas** (`producto/`, `arquitectura/`, `costos/`, `gestion/`), previendo que entre más documentación. Nombres de archivo sin cambios, solo la carpeta; enlaces cruzados actualizados |
| 1.2 | 2026-08-19 | **RF-25 nuevo: ejecución opcional de las pruebas generadas en un sandbox del navegador** (Pyodide para Python, runner propio compatible con Vitest para JS/TS), nunca en el servidor. Revierte parcialmente la decisión original de "no ejecutar nada" — el sandbox de *servidor* (Docker/Firecracker/E2B/Judge0) sigue descartado, el de *cliente* no. RNF-03, RF-12 y la lista de alcance del PRD se actualizan; nueva sección 2.14 en 02-arquitectura.md; Fase 1 de 04-plan.md gana esta pieza, reusada en Fase 2 para automatizar la corrida del conjunto de evaluación. Investigación de alternativas de sandboxing hecha antes de decidir |
| 1.3 | 2026-08-19 | **Calendario de sprints fijado.** Sprint 1 arranca hoy (2026-08-19), termina 2026-09-01; tabla completa de fechas por sprint hasta el deadline (2026-12-01) en 04-plan.md, con 6 días de colchón al final. Deja de estar pendiente de Sprint Planning el reparto de fechas por fase |

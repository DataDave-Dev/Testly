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
| [00-idea.md](00-idea.md) | Idea original y decisiones tomadas durante el mapeo |
| [01-prd.md](01-prd.md) | Objetivos, justificación, alcance y requisitos |
| [02-arquitectura.md](02-arquitectura.md) | Stack, flujo, prompt, modelo de datos, seguridad |
| [03-costos.md](03-costos.md) | Costo por generación y proyección mensual |
| [04-plan.md](04-plan.md) | Fases, entregables y riesgos |
| [05-proveedor-llm.md](05-proveedor-llm.md) | Opciones gratuitas y de pago para el modelo, con datos verificados |
| [06-estructura-carpetas.md](06-estructura-carpetas.md) | Cinco formas de organizar `src/`, comparadas |
| [07-metodologia.md](07-metodologia.md) | Metodología de desarrollo (Scrum), equipo con matrícula, roles, sprints, ceremonias |
| [08-limites.md](08-limites.md) | Fecha límite, retención y borrado de datos, acceso, tope de salida del modelo, tabla de límites técnicos |
| [09-carta-patrocinadora.md](09-carta-patrocinadora.md) | Carta patrocinadora (project charter): beneficio, entregables, medidas de éxito, fechas, roles pendientes de asignar |

## Historial de versiones

| Versión | Fecha | Cambio |
|---|---|---|
| 0.1 | 2026-08-17 | Producto planteado como analizador: explicación, errores y estilo |
| 0.2 | 2026-08-18 | **Cambio de producto.** Ahora genera pruebas unitarias y explica el criterio de cada caso. Cambian el prompt, el modelo de datos, los criterios de éxito y el costo (aproximadamente el doble) |
| 0.3 | 2026-08-18 | Precios y modelos verificados contra la documentación oficial. Sonnet 5 quedó en $2/$10 de forma permanente; se corrigen los mínimos de caché por modelo y se documenta que Haiku 4.5 no acepta `effort`. Recomendación de partida: Sonnet 5 |
| 0.4 | 2026-08-18 | **Pivotes de arquitectura y cierre de decisiones económicas.** Hosting: Cloudflare Workers en vez de Vercel (sin límite de tiempo de pared para el streaming). Base de datos: Turso en vez de Supabase, con Better Auth (Lucia está descontinuada). Login pasa de obligatorio a **opcional**: generación libre con cuota por IP, sesión solo para historial y cuota mayor. Se agregan middleware (control de acceso) y validación por aserciones de framework (RF-20). Se cierran las decisiones pendientes: el equipo paga la API ($20 USD), modelo Sonnet 5, cuota de 10/periodo con sesión y 3/día sin sesión. Con eso, [05-proveedor-llm.md](05-proveedor-llm.md) queda archivado como referencia, no como plan activo |
| 0.5 | 2026-08-18 | Se agrega [07-metodologia.md](07-metodologia.md): metodología Kanban sobre las fases de 04-plan.md, equipo con matrícula, reparto sugerido por área técnica |
| 0.6 | 2026-08-18 | **Cambio de metodología a Scrum** en 07-metodologia.md (decisión del equipo, reemplaza la propuesta de Kanban). **Reducción de alcance de lenguajes**: el MVP cubre Python y JavaScript/TypeScript; Java y C/C++ quedan fuera, documentados como trabajo futuro en 00-idea.md, 01-prd.md (secciones 2 y 5), 02-arquitectura.md (sección 5.2), 03-costos.md, 04-plan.md (Fase 0 y tabla de riesgos) y 05-proveedor-llm.md (archivado, ajustado solo donde la conclusión cambiaba) |
| 0.7 | 2026-08-18 | Se agrega [08-limites.md](08-limites.md). **Fecha límite fijada: 2026-12-01** (~7 sprints disponibles, riesgo de calendario documentado en 04-plan.md y 07-metodologia.md). **RF-23 nuevo**: eliminación de cuenta por soft delete, detalle de implementación con el hook `beforeDelete` de Better Auth en 02-arquitectura.md sección 2.13. Acceso abierto a cualquiera confirmado. Tope de salida del modelo formalizado (`LLM_MAX_OUTPUT_TOKENS=16000`) |
| 0.8 | 2026-08-19 | Se agrega [09-carta-patrocinadora.md](09-carta-patrocinadora.md): carta sponsor original adaptada al proyecto actual, roles Scrum sin persona asignada dejado pendiente a propósito |
| 0.9 | 2026-08-19 | **Nombre del proyecto fijado: Testly.** Se propaga a todos los documentos que se referían a él sin nombre |
| 1.0 | 2026-08-19 | Esta carpeta pasa a versionarse en el repositorio: se retira la nota que la excluía por `.gitignore` |

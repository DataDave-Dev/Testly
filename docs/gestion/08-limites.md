# Límites del proyecto — Testly

Fecha: 2026-08-18
Versión: 0.1

Este documento junta los límites que definen el proyecto: los que ya venían
documentados por pieza (cuota, presupuesto, tamaño de código) y los que se
cerraron en esta ronda (fecha límite, retención de datos, acceso, tope de
salida del modelo). No repite el detalle de cada uno — enlaza a la fuente.

## 1. Fecha límite

**2026-12-01. Proyecto terminado en su totalidad, sin excepción.**

Desde hoy (2026-08-18) son ~15 semanas disponibles. Con sprints de 2 semanas
([07-metodologia.md](07-metodologia.md)), son **~7 sprints** para las 6 fases
de [04-plan.md](04-plan.md). Es un calendario ajustado — la Fase 1 sola se
estimó en 2 sprints — documentado como riesgo Alto en 04-plan.md, con el orden
de recorte ya decidido: Fase 4 (interfaz) primero, nunca la Fase 2
(calibración) ni la Fase 5 (prueba con usuarios), porque esas dos sostienen
los criterios de éxito del PRD.

## 2. Retención y borrado de datos

**Sin política general de borrado.** El código pegado por el estudiante y el
resultado generado se conservan indefinidamente en `generations` mientras la
cuenta exista. Es una decisión de producto, no un descuido: sostiene el
argumento de la sección 10 del PRD (un profesor puede pedir ver el historial
si sospecha uso indebido) y la auditoría de gasto de RNF-08.

**Excepción: eliminación de cuenta.** El usuario puede eliminar su cuenta
(RF-23). Ahí sí aplica un límite: la eliminación es **soft delete**, nunca
borrado físico.

- Se marca `deleted_at` en `user` y se anonimizan email y nombre.
- Las filas de `generations` de esa cuenta **no se tocan**: siguen existiendo,
  ahora huérfanas de una cuenta anonimizada.
- El usuario no puede volver a iniciar sesión con esa cuenta.

Detalle de implementación (hook `beforeDelete` de Better Auth) en
[02-arquitectura.md](../arquitectura/02-arquitectura.md), sección 2.13.

## 3. Acceso

**Abierto a cualquiera, sin restricción de dominio ni institución.** No se
limita a correo `@uanl.mx` ni a ningún otro dominio. Esto ya era el
comportamiento documentado en RF-03 ([01-prd.md](../producto/01-prd.md)) — este punto lo
deja explícito como decisión, no como omisión.

## 4. Tope de salida del modelo

**`LLM_MAX_OUTPUT_TOKENS=16000`.** Si una respuesta se corta antes de cerrar
el bloque cercado del archivo de pruebas, la descarga sale truncada sin
avisar — es la advertencia ya anotada en
[02-arquitectura.md](../arquitectura/02-arquitectura.md), sección 5.3, y en
[03-costos.md](../costos/03-costos.md), sección 8. Este documento lo formaliza como
límite de producto, no solo como detalle de la petición al modelo. Registrar
el `stop_reason` de cada llamada sigue siendo obligatorio para detectar
truncamiento.

## 5. Límites técnicos ya documentados

Referencia rápida; el detalle y la fuente de cada uno vive en su documento:

| Límite | Valor | Fuente |
|---|---|---|
| Tamaño de código de entrada | 12,000 caracteres (~3,000 tokens) | RF-06, [01-prd.md](../producto/01-prd.md) |
| Cuota con sesión | 10 generaciones / 30 días | RF-17 |
| Cuota sin sesión | 3 / día por IP | RF-17 |
| Tiempo de respuesta | <45s en 90% de casos | RNF-01 |
| Presupuesto | $20 USD, todo el semestre | [03-costos.md](../costos/03-costos.md) |
| CPU por petición (Cloudflare gratis) | 10ms | [02-arquitectura.md](../arquitectura/02-arquitectura.md) 2.8 |
| Bundle comprimido (Cloudflare gratis) | 3MB | 02-arquitectura.md 2.11 |
| Turso gratis | 5GB, 500M lecturas/mes, 10M escrituras/mes | 02-arquitectura.md 2.4 |
| Lenguajes soportados | Python, JavaScript/TypeScript (MVP) | 01-prd.md sección 5 |

## 6. Pendiente

- **Horas/semana reales por persona.** Sin esto, el Sprint Planning de
  [07-metodologia.md](07-metodologia.md) reparte tareas sin saber cuánto cabe
  de verdad en un sprint de 2 semanas. Es una decisión del equipo, no algo que
  se pueda inferir de la documentación.

## Historial de versiones

| Versión | Fecha | Cambio |
|---|---|---|
| 0.1 | 2026-08-18 | Primera versión: fecha límite (2026-12-01), política de retención y soft delete de cuenta (RF-23 nuevo), acceso abierto confirmado, tope de salida del modelo formalizado, tabla de referencia de límites técnicos ya documentados |

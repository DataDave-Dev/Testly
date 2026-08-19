# Plan de trabajo — Testly

Fecha: 2026-08-18
Versión: 0.8

**Fecha límite: 2026-12-01.** Proyecto terminado en su totalidad, sin
excepción. Desde hoy (2026-08-18) son ~15 semanas, unos **7 sprints de 2
semanas** (ver [07-metodologia.md](07-metodologia.md)) para las 6 fases de
abajo. Es ajustado: la Fase 1 sola se estimó en 2 sprints. Si a mitad de
camino el ritmo no alcanza, se recorta la Fase 4 (interfaz, ver sección de esa
fase) antes que la Fase 2 (calibración) o la Fase 5 (prueba con usuarios), que
son las que sostienen los criterios de éxito del PRD.

## Calendario de sprints

**Fijado el 2026-08-19**, con el Sprint 1 arrancando ese mismo día (mapeo
exacto de 1 fase por sprint, salvo la Fase 1 que se estimó en 2). Sprints de
2 semanas, sin hueco entre uno y el siguiente.

| Sprint | Fechas | Fase |
|---|---|---|
| 1 | 2026-08-19 → 2026-09-01 | Fase 0 — Preparación |
| 2 | 2026-09-02 → 2026-09-15 | Fase 1 — Núcleo de la generación (1/2) |
| 3 | 2026-09-16 → 2026-09-29 | Fase 1 — Núcleo de la generación (2/2) |
| 4 | 2026-09-30 → 2026-10-13 | Fase 2 — Calibración |
| 5 | 2026-10-14 → 2026-10-27 | Fase 3 — Cuentas opcionales y persistencia |
| 6 | 2026-10-28 → 2026-11-10 | Fase 4 — Interfaz |
| 7 | 2026-11-11 → 2026-11-24 | Fase 5 — Prueba con usuarios y cierre |
| — | 2026-11-25 → 2026-12-01 | Colchón de 6 días antes del deadline, sin sprint asignado |

Si un sprint se atrasa, el criterio de recorte sigue siendo el de arriba:
Fase 4 primero, nunca la Fase 2 ni la Fase 5. El colchón de 6 días no alcanza
para absorber un sprint completo perdido — es margen de cierre, no un sprint
8 disfrazado.

## Fases

### Fase 0 — Preparación

- Confirmar equipo y reparto de responsabilidades
- ~~Conseguir acceso y presupuesto de la API del modelo~~ — **resuelto**: el
  equipo paga, $20 USD ya disponibles (ver [03-costos.md](../costos/03-costos.md))
- ~~Definir límite de cuota por usuario~~ — **resuelto**: 10 por 30 días con
  sesión, 3/día por IP sin sesión
- Armar el conjunto de evaluación: **20 funciones** en los lenguajes del MVP
  (Python y JavaScript/TypeScript), cada una con su lista documentada de casos
  que las pruebas deberían cubrir (camino feliz, valores límite, entradas
  inválidas)
- Incluir a propósito en ese conjunto tres o cuatro fragmentos **difíciles de
  probar**: un script sin funciones, algo que dependa de la hora del sistema,
  algo que lea un archivo. Sirven para verificar RF-09, que el sistema sepa
  decir "esto no es apto" en lugar de inventar pruebas

**Entregable:** conjunto de evaluación y presupuesto confirmado.

El conjunto de evaluación va primero a propósito: sin él no se puede comparar
modelos ni niveles de `effort`, y esas dos decisiones definen el costo del
proyecto entero. La lista de casos esperados por función es la parte laboriosa
y no se puede improvisar en la Fase 2.

### Fase 1 — Núcleo de la generación

- Proyecto Astro con adapter de Cloudflare (`@astrojs/cloudflare`)
- **Prueba de streaming largo desplegada**: un endpoint que transmita 60 segundos
  y llegue completo al navegador desde el dominio de Cloudflare, no desde
  `localhost`. Cloudflare no debería cortarlo por tiempo de pared, pero hay que
  verificarlo igual: es la prueba que antes habría fallado en Vercel gratis
- Aserciones de validación por framework (ver
  [02-arquitectura.md](../arquitectura/02-arquitectura.md), sección 2.11), con sus pruebas
  unitarias
- `frameworks.ts` y `extraerTest.ts` con sus pruebas unitarias
- Endpoint `/api/generar` funcionando de extremo a extremo, **sin sesión**
- **Protección del endpoint desde el día uno, no en la Fase 3**: regla de Rate
  Limiting de Cloudflare por IP + Cloudflare Turnstile + tabla `anon_quota` con
  el tope de 3/día. Como el flujo sin cuenta es el flujo principal del
  producto, no hay una fase posterior de "ya se puede exponer con seguridad":
  está expuesto desde que existe
- Primera versión del prompt de sistema
- Pantalla mínima: pegar código, elegir lenguaje y framework, ver el resultado
  en streaming, descargar el archivo
- **Sandbox de ejecución del lado del cliente (RF-25):** Pyodide (Python) +
  runner propio compatible con Vitest (JS/TS), en iframe/Worker aislado,
  botón "Correr pruebas" bare-bones — nunca en el servidor. Ver
  [02-arquitectura.md](../arquitectura/02-arquitectura.md) sección 2.14. Se
  construye aquí porque la Fase 2 lo reusa para automatizar su corrida

**Entregable:** un archivo de pruebas real, generado desde el navegador en
producción y descargado.

Este es el punto de mayor riesgo técnico. Va temprano por eso: si las pruebas
generadas no sirven, todo lo demás sobra.

La descarga entra en la Fase 1 y no en la de interfaz porque es la que verifica
que la extracción del bloque cercado funciona contra salida real del modelo.
Es lógica, no adorno.

### Fase 2 — Calibración

El modelo de producción ya está decidido: **Sonnet 5** (ver
[03-costos.md](../costos/03-costos.md), sección 6.1). Esta fase ya no compara modelos
para elegir uno; mide `effort` sobre el modelo decidido, y opcionalmente reúne
un dato de respaldo con Opus 5 por si Sonnet 5 no alcanza el criterio de
validez más adelante.

- Correr el conjunto de evaluación contra Sonnet 5 en `low`, `medium` y `high`
- **Opcional, recomendado por ser casi gratis vía Batch API:** correr también
  el conjunto contra Opus 5 (un solo nivel de `effort` basta, `medium`). No es
  para elegir modelo — es tener un número real en la mano si hace falta subir
  de modelo más adelante en vez de tener que medir de nuevo a mitad de semestre
- **Usar la Batch API**: 50% de descuento y no hay prisa. Con Sonnet 5 en los
  tres niveles más un pase de Opus 5, la fase completa cuesta unos $3-4, muy
  por debajo del presupuesto de $20
- **Correr las pruebas generadas con el sandbox de la Fase 1** (RF-25, en modo
  headless) y registrar cuántas corren sin errores de sintaxis, import o
  nombre (criterio de validez del PRD) — ya no se mide completamente a mano
- Revisar a mano cuántas pruebas son tautológicas
- Verificar que los fragmentos "difíciles de probar" reciben la respuesta de
  RF-09 y no pruebas inventadas
- Elegir `effort` por defecto, con los números en la mano
- Iterar el prompt de sistema con base en lo que falle
- Actualizar [03-costos.md](../costos/03-costos.md) con los tokens medidos y el
  `stop_reason` de cada llamada

**Entregable:** tabla de validez y costo por nivel de `effort`, y decisión
tomada. Si Sonnet 5 no llega a 17/20, el dato de Opus 5 ya está medido y no hay
que empezar una nueva ronda de calibración para decidir el cambio.

Esta fase sigue siendo más pesada que explicar código en vez de generarlo: hay
que correr 20 archivos de pruebas por cada nivel de `effort` evaluado.
Conviene automatizar la corrida con un script desde el inicio, no evaluar a
mano combinación por combinación.

### Fase 3 — Cuentas opcionales y persistencia

- Turso: esquema (`generations`, `anon_quota`, tablas de Better Auth)
- Better Auth: sesión por enlace mágico (correo, sin contraseña), inicio y
  cierre de sesión
- Aislamiento manual por `user_id` en `src/lib/db/generations.ts` (sin RLS,
  ver [02-arquitectura.md](../arquitectura/02-arquitectura.md), sección 2.4)
- Guardado de cada generación con sus tokens, **solo si hay sesión**
- Historial: lista, detalle, eliminación
- Cuota por usuario con sesión, validada en el servidor
- Manejo de errores visible (RF-19: fallo de API no descuenta cuota)

**Entregable:** flujo completo con sesión opcional e historial para quien la
crea. El flujo sin sesión, que ya funcionaba desde la Fase 1, no cambia.

### Fase 4 — Interfaz

- **Archivo de tokens primero**: paleta, tipografía (incluida la monoespaciada
  elegida a propósito) y escala de espaciado configuradas en el tema de
  Tailwind, antes del primer componente
- Diseño de las pantallas
- **Landing** (RF-21): qué hace la plataforma, teoría breve de pruebas
  unitarias, al menos un ejemplo, llamada a probarlo sin pedir cuenta
- Editor con selectores de lenguaje y framework, y aviso sobre credenciales
- Presentación del resultado: casos primero, archivo después
- Bloque de código con copiar y descargar, y el aviso de RF-12 de que las
  pruebas no fueron ejecutadas
- Estados: cargando, código no apto para pruebas, cuota agotada, fallo de API
- Responsive desde 375px

**Entregable:** producto presentable.

### Fase 5 — Prueba con usuarios y cierre

- Prueba con al menos 10 estudiantes ajenos al equipo
- Encuesta breve: no solo si les gustó, sino **si podrían escribir el siguiente
  archivo de pruebas por su cuenta**. Es el objetivo declarado del producto y
  hay que medirlo
- Corrección de lo que salga
- Documentación final y presentación

**Entregable:** resultados de la prueba y documentación completa.

## Riesgos

| Riesgo | Impacto | Mitigación |
|---|---|---|
| Las pruebas generadas no compilan o no pasan | Alto | Es el riesgo central. Se mide en Fase 2 con el criterio de validez; si un modelo no llega a 17 de 20, se sube de modelo o de `effort` aunque cueste más |
| El modelo genera pruebas tautológicas que se ven bien | Alto | Prohibición explícita en el prompt; revisión manual en Fase 2. No se detecta solo: hay que buscarlo |
| El costo real supera el estimado | Alto | Medir en Fase 2 antes de comprometer presupuesto; cuota por usuario desde Fase 3. Con Sonnet 5 y cuota de 10, harían falta 83 usuarios agotando su cuota por completo para tocar los $20 disponibles |
| Abuso del flujo sin cuenta (scripts contra el límite de 3/día por IP) | Medio | Tres capas desde la Fase 1: regla de Rate Limiting de Cloudflare, Turnstile, tabla `anon_quota`. Ninguna sola alcanza; juntas sí. Si aun así se abusa, la salida es bajar el tope anónimo o exigir sesión siempre |
| Sonnet 5 no alcanza el criterio de validez en Python o JavaScript/TypeScript | Medio | Modelo ya decidido (Sonnet 5), pero la Fase 2 mide también Opus 5 como respaldo (sección Fase 2) para no tener que recalibrar a mitad de semestre si hace falta subir de modelo |
| El límite de duración de función del hosting corta el streaming | Bajo (era Alto con Vercel) | Cloudflare Workers no impone límite de tiempo de pared en peticiones HTTP; el límite es de CPU y esperar al modelo es I/O. Verificar igual con la prueba de 60 s de la Fase 1. Si Cloudflare no funciona por otra razón, Vercel es plan B con el riesgo original |
| La respuesta se trunca por `max_tokens` y el archivo descargado queda a medias | Medio | Registrar `stop_reason` desde la Fase 1; si aparece truncamiento, subir el límite o acotar los casos pedidos |
| Se usa para hacer la tarea de escribir pruebas | Medio | Reconocido de frente en el PRD, sección 10. La explicación no es omitible y el código pegado queda en el historial |
| La API falla o cambia durante el semestre | Medio | Modelo configurable por variable de entorno; manejo de errores explícito |
| El equipo no domina Astro con SSR | Medio | Fase 1 corta y temprana; si el adapter da problemas, se detecta con semanas de margen |
| Alcance que crece (cobertura, mocks, ejecución de servidor) | Medio | La lista de "fuera del MVP" del PRD sigue siendo la defensa para lo que queda fuera. "Ejecutar las pruebas" ya no es un riesgo abierto: se decidió como sandbox de navegador con costo $0 (RF-25, ver 02-arquitectura.md sección 2.14) — la tentación que sigue viva es escalar a ejecución de servidor (Docker/Firecracker/E2B), eso sí sigue fuera del MVP |
| Turso o Better Auth cambian condiciones del plan gratuito durante el semestre | Bajo | El consumo del proyecto está muy por debajo del margen gratuito de Turso (sección 2.4 de arquitectura); verificar igual antes de la presentación |
| Agregar Java o C/C++ más adelante resulta más caro de calibrar de lo previsto | Bajo | Quedan fuera del MVP desde [01-prd.md](../producto/01-prd.md), sección 5. Si se agregan después de la Fase 5, repiten la Fase 2 (conjunto de evaluación propio, medición de validez) antes de anunciarse como soportados |
| Las 6 fases no caben en los ~7 sprints disponibles antes del 2026-12-01 | Alto | Fecha límite fija, no movible. Si un sprint se atrasa, se recorta primero la Fase 4 (interfaz), nunca la Fase 2 (calibración) ni la Fase 5 (prueba con usuarios) |

## Dependencias externas

| Dependencia | Riesgo si falla |
|---|---|
| API del modelo (Anthropic) | Bloqueante: es el núcleo del producto |
| Turso | Medio: historial y sesiones. El flujo principal (generar sin cuenta) sigue vivo si falla; solo se pierde historial mientras se recupera |
| Better Auth | Bajo: sin él, el producto sigue funcionando en modo anónimo, solo sin historial |
| Cloudflare Turnstile | Medio: si falla, el flujo sin cuenta queda sin la segunda capa de protección contra abuso hasta restablecerlo |
| Hosting con soporte SSR y funciones largas | Medio: varias alternativas equivalentes |

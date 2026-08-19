# Costos de operación — Testly

Fecha: 2026-08-18
Versión: 0.6
Precios verificados contra la documentación oficial en esta fecha.
**Verificar antes de presupuestar**: los precios de los modelos cambian.

## 1. Por qué este documento existe

Cada generación es una llamada de pago a una API externa. A diferencia de un
proyecto escolar típico, aquí el costo crece con el uso. Sin límites, un solo
usuario con un script puede consumir el presupuesto del semestre en una tarde.

Por eso el login es obligatorio desde el MVP y por eso se guardan los tokens de
cada llamada.

**Restricción del proyecto: minimizar el costo.** Es un proyecto escolar, no un
producto con ingresos. Este documento está escrito con esa meta, no con la de
maximizar calidad a cualquier precio. Donde hay que elegir, se elige lo barato
y se documenta qué se sacrifica.

## 2. Cómo se cobra

Se paga por token, con precios distintos para entrada y salida:

- **Entrada**: todo lo que se le envía al modelo (prompt de sistema + código).
- **Salida**: todo lo que el modelo genera, incluido su razonamiento interno.

La salida cuesta 5 veces más que la entrada en todos los modelos. Eso define
dónde está la palanca: **controlar el largo de la respuesta importa mucho más
que recortar el prompt**.

En este producto la respuesta es larga por diseño: un archivo de pruebas con
cinco u ocho casos, más el razonamiento de cada uno.

## 3. Modelos disponibles y precios

Precios por millón de tokens (USD), a 2026-08-18:

| Modelo | ID | Entrada | Salida | Lectura de caché |
|---|---|---|---|---|
| Claude Opus 5 | `claude-opus-5` | $5.00 | $25.00 | $0.50 |
| Claude Sonnet 5 | `claude-sonnet-5` | $2.00 | $10.00 | $0.20 |
| Claude Haiku 4.5 | `claude-haiku-4-5` | $1.00 | $5.00 | $0.10 |

Claude Fable 5 ($10/$50) queda descartado sin discusión: cuesta el doble que
Opus 5 y este proyecto no tiene ningún problema que lo justifique.

**Nota sobre el precio de Sonnet 5.** Su precio de $2/$10 se anunció como
introductorio hasta el 2026-08-31, con un aumento programado a $3/$15 el 1 de
septiembre. **Ese aumento fue cancelado**: $2/$10 es ahora el precio estándar.
No hay que presupuestar un aumento en septiembre.

### 3.1 Diferencias que no son de precio

No son tres versiones del mismo modelo con distinto costo. Hay diferencias
funcionales que afectan el diseño:

| | Opus 5 | Sonnet 5 | Haiku 4.5 |
|---|---|---|---|
| Parámetro `effort` | Sí | Sí | **No** |
| Razonamiento adaptativo | Sí | Sí | **No** |
| Salida máxima | 128k tokens | 128k tokens | 64k tokens |
| Contexto | 1M tokens | 1M tokens | 200k tokens |
| Mínimo cacheable | 512 tokens | 1,024 tokens | **4,096 tokens** |

Las dos casillas en negrita son las que importan para este proyecto y se
explican en las secciones 6.1 y 6.4.

## 4. Supuestos del cálculo

| Concepto | Valor | Nota |
|---|---|---|
| Prompt de sistema | ~1,800 tokens | Instrucciones pedagógicas, reglas por framework, prohibiciones. Estable |
| Código del estudiante | ~800 tokens | Equivale a unas 100 líneas |
| **Entrada total** | **~2,600 tokens** | |
| Sección de casos | ~500 tokens | La parte pedagógica |
| Archivo de pruebas | ~900 tokens | Cinco a ocho casos con sus aserciones |
| Cómo correrlo | ~100 tokens | |
| Razonamiento interno | ~700 tokens | Acotado. Ver advertencia de la sección 8 |
| **Salida total** | **~2,200 tokens** | |

Son una estimación de partida, no una medición. Ver sección 8.

## 5. Costo por generación

| Modelo | Sin caché | Con caché del prompt | En pesos (18.5 MXN/USD) |
|---|---|---|---|
| Opus 5 | $0.068 | **$0.060** | $1.11 |
| Sonnet 5 | $0.027 | **$0.024** | $0.44 |
| Haiku 4.5 | $0.014 | $0.014 (no cachea, ver 6.4) | $0.25 |

### Lo que realmente cuesta el proyecto

Antes de la proyección hipotética: **el semestre completo son unas 600
generaciones**, no 1,000 al mes. El PRD pide que diez estudiantes ajenos al
equipo generen al menos una vez; el resto es calibración, desarrollo y demo.

| Modelo | Semestre completo (~600) | En pesos |
|---|---|---|
| Haiku 4.5 | $8.40 | ~$155 |
| Sonnet 5 | $14.40 | ~$266 |
| Opus 5 | $40.80 | ~$755 |

**Decidido:** el equipo paga de su bolsa, con $20 USD ya disponibles en cuenta.
Sonnet 5 deja **$5.60 de margen** (~233 generaciones extra) sobre el costo
estimado del semestre. Con eso, perseguir capa gratuita deja de ser necesario;
ver [05-proveedor-llm.md](05-proveedor-llm.md) sección 7 para el cierre de esa
investigación.

### Proyección mensual si el producto creciera

| Generaciones/mes | Opus 5 | Sonnet 5 | Haiku 4.5 |
|---|---|---|---|
| 500 | $30 | $12 | $7 |
| 1,000 | $60 | $24 | $14 |
| 3,000 | $180 | $72 | $41 |
| 5,000 | $300 | $120 | $68 |

Referencia: 100 estudiantes activos haciendo 10 generaciones al mes son 1,000
generaciones/mes.

**Lectura práctica.** A 1,000 generaciones mensuales, Opus 5 cuesta $60 y
Sonnet 5 cuesta $24. Esa diferencia —unos $700 pesos mensuales— es la decisión
económica del proyecto, y es la razón de ser de la Fase 2.

## 6. Palancas de reducción, en orden de impacto

### 6.1 Elección de modelo (impacto alto)

Haiku 4.5 cuesta una quinta parte de Opus 5. Pero la elección no es solo de
precio, porque **Haiku 4.5 no acepta el parámetro `effort` ni el razonamiento
adaptativo**. Consecuencias concretas:

- Si se elige Haiku, la palanca de la sección 6.2 **deja de existir**. El costo
  se controla solo por el largo del prompt y por `max_tokens`.
- Generar código correcto se beneficia del razonamiento previo, que es donde el
  modelo verifica que las aserciones correspondan al código real. Haiku no lo
  tiene en su forma adaptativa.

La duda concreta: **generar código correcto es más exigente que explicar
código.** Un modelo pequeño puede describir bien un bucle y aun así producir un
archivo con un import equivocado o una aserción sobre una función inexistente.
El criterio de validez del PRD (17 de 20 corren sin errores) es exactamente la
medición que decide esto.

**Decidido: Sonnet 5.** Balance entre calidad de análisis, potencia y precio;
conserva `effort` y razonamiento adaptativo, y la documentación oficial lo
describe como la mejor combinación de velocidad e inteligencia para este tipo
de tarea. Con presupuesto propio de $20 y un costo de semestre de $14.40, ya no
hace falta elegir el modelo más barato posible — Haiku deja de evaluarse como
candidato a producción.

**Una recomendación, no una condición:** aunque el modelo de producción ya está
decidido, vale la pena mantener a **Opus 5 dentro de la matriz de la Fase 2**
como medición de respaldo, no como candidato a elegir. Correrlo cuesta unos $2
extra por la Batch API (sección 6.5) y deja un número real en la mano si Sonnet
5 no alcanza el criterio de validez en Python o JavaScript/TypeScript —riesgo
ya anotado en el plan de trabajo—, en vez de tener que volver a medir a mitad
de semestre. Es
opcional: si prefieren simplificar la Fase 2, correrla solo contra Sonnet 5 en
los tres niveles de `effort` también es válido.

Dejar el modelo en `LLM_MODEL` para poder cambiarlo sin tocar código, aunque el
valor por defecto ya no cambie salvo que la Fase 2 lo obligue.

### 6.2 Nivel de `effort` (impacto alto — solo Opus 5 y Sonnet 5)

`effort` controla cuánto razona el modelo antes de responder, y ese
razonamiento se cobra como salida. Los niveles son `low`, `medium`, `high`,
`xhigh` y `max`. **El valor por defecto es `high`**: si no se fija
explícitamente, se está pagando el nivel alto sin haberlo decidido.

Para minimizar costo hay que **fijarlo explícitamente y hacia abajo**. Empezar
en `medium` y probar `low` contra el conjunto de evaluación. Bajar de `high` a
`medium` recorta una parte notable de los tokens de salida.

El riesgo: en generación de código, `low` puede producir pruebas que no
compilan. Se mide en la Fase 2 comparando costo contra el criterio de validez,
no contra la impresión de calidad.

Este parámetro no existe en Haiku 4.5.

### 6.3 Número de casos generados (impacto medio-alto)

Palanca directa: el prompt puede acotar el rango de casos (por ejemplo, de
cuatro a seis en vez de "los que valgan la pena"). Cada caso son unos 150
tokens de salida entre el código y su explicación.

Bajar de ocho casos a cinco recorta cerca del 20% de la salida. El riesgo es
recortar justo el caso borde interesante, que es la parte pedagógica. Punto
medio razonable: acotar el rango y exigir que se prioricen las tres familias
obligatorias por encima del volumen.

### 6.4 Caché del prompt de sistema (impacto medio, con una trampa)

El prompt de sistema no cambia entre peticiones, así que puede cachearse. La
lectura de caché cuesta la décima parte de la entrada normal; la escritura
cuesta 1.25 veces. Con dos lecturas ya salió a cuenta.

**La trampa: cada modelo tiene un mínimo de tokens por debajo del cual no
cachea, y no avisa.** No hay error: simplemente devuelve cero tokens de caché.

| Modelo | Mínimo cacheable | ¿Cachea el prompt de ~1,800 tokens? |
|---|---|---|
| Opus 5 | 512 | Sí |
| Sonnet 5 | 1,024 | Sí |
| Haiku 4.5 | 4,096 | **No** |

Con Opus 5 o Sonnet 5 el ahorro es de alrededor del 12% del total. Es real pero
modesto, porque el costo lo domina la salida.

**Consecuencia contraintuitiva si se elige Haiku 4.5:** ampliar el prompt de
sistema de 1,800 a 4,096 tokens lo vuelve cacheable y **baja** el costo por
generación de $0.0136 a $0.0122, alrededor de un 10%. Es el único caso en este
proyecto donde escribir un prompt más largo sale más barato. Si el equipo se
queda con Haiku, conviene aprovechar esos tokens extra en más ejemplos de
pruebas bien escritas, no en relleno.

Verificar siempre con el campo `cache_read_input_tokens` de la respuesta: si
sale cero en peticiones repetidas, el caché no está funcionando.

### 6.5 API de lotes para la Fase 2 (impacto alto, pero solo en la calibración)

La Batch API tiene **50% de descuento** en entrada y salida. No sirve para el
producto —es asíncrona y el producto necesita streaming— pero sí para correr el
conjunto de evaluación, que no es sensible a la latencia.

Costo de la Fase 2 completa (20 fragmentos × 3 modelos × 3 niveles de `effort`
= 180 llamadas):

| | Precio normal | Con Batch API |
|---|---|---|
| Todo en Sonnet 5 | $4.90 | **$2.45** |
| Peor caso, todo en Opus 5 | $12.20 | **$6.10** |

**La fase de calibración cuesta menos de diez dólares.** No es un motivo para
recortarla ni para evaluar menos combinaciones.

### 6.6 Cuota por usuario (control, no reducción)

No baja el costo por generación; pone techo al gasto total:

```
gasto máximo = usuarios registrados × N × costo por generación
```

**Decidido: 10 generaciones por periodo (30 días) para usuario con sesión.**
Con Sonnet 5 a $0.024, el techo es predecible:

```
gasto máximo = usuarios registrados × 10 × $0.024 = usuarios registrados × $0.24
```

| Usuarios registrados maximizando cuota | Gasto |
|---|---|
| 10 (el mínimo que pide el PRD) | $2.40 |
| 50 | $12 |
| 83 | $19.92 — el techo teórico de los $20 disponibles |

Ese último renglón es el que importa: **83 usuarios tendrían que agotar su
cuota por completo** en un mismo periodo de 30 días para tocar el presupuesto.
El PRD solo pide diez estudiantes ajenos al equipo generando una vez cada uno.
El margen es amplio.

## 7. Configuración decidida

```
LLM_MODEL=claude-sonnet-5     # decidido; effort es lo único que ajusta la Fase 2
LLM_EFFORT=medium             # fijarlo: el default es 'high' y cuesta más
LLM_MAX_OUTPUT_TOKENS=16000   # tope de salida; ver 08-limites.md
QUOTA_PER_PERIOD=10           # generaciones por 30 días, con sesión
QUOTA_ANONIMA_DIARIA=3        # sin sesión, por IP (ver 02-arquitectura.md)
MAX_CODE_CHARS=12000          # ~3,000 tokens de código como máximo
```

`LLM_EFFORT` es lo único de esta tabla que la Fase 2 todavía puede mover: se
mide `low` y `medium` contra el criterio de validez y se deja el más bajo que
lo cumpla.

## 8. Qué medir antes de presupuestar en serio

1. **Contar tokens reales.** El endpoint `count_tokens` mide el prompt exacto
   sin gastar en generación. Sirve para fijar el número de entrada.
2. **Registrar `usage` de las primeras llamadas reales.** Cada respuesta trae
   los tokens consumidos, incluidos `cache_read_input_tokens`. La tabla
   `generations` ya los guarda (RNF-08). Con 50 generaciones reales se tiene un
   promedio confiable.
3. **Ajustar las tablas de este documento con los números medidos.**

**Advertencia sobre la salida.** Si el modelo razona mucho antes de responder,
esos tokens se cobran como salida y pueden superar por mucho los 700 supuestos.
Si al medir aparecen respuestas de 5,000 o 6,000 tokens de salida, el costo real
es del orden de dos a tres veces el estimado. Bajar `effort` es la respuesta.

**Advertencia sobre `max_tokens`.** Si una respuesta se corta, el bloque cercado
del archivo de pruebas nunca cierra y el botón de descarga entregaría un archivo
truncado sin avisar. Hay que registrar el `stop_reason` de cada llamada: si
aparece `max_tokens`, es un bug de producto antes que un tema de costo.

## 9. Decisiones

- [x] ¿Quién paga la API y con qué presupuesto mensual? El equipo, $20 USD ya
      disponibles. No es mensual: es el presupuesto para todo el semestre.
- [x] ¿Cuál es el límite de generaciones por usuario por periodo? 10, con
      sesión. 3/día por IP, sin sesión.
- [x] ¿Sonnet 5 o Haiku 4.5 por defecto? Sonnet 5, decidido. La Fase 2 mide
      `effort`, no modelo — con la recomendación opcional de correr también
      Opus 5 como respaldo medido (sección 6.1).
- [ ] ¿Se acota el número de casos generados, y a qué rango? Sigue abierta.

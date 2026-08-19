# Elección de proveedor del modelo — Testly

Fecha: 2026-08-18
Versión: 1.2 — **cerrado**: el equipo paga la API directamente (sección 7).
Nota: el alcance de lenguajes se redujo después de escribir este documento —
el MVP cubre Python y JavaScript/TypeScript, Java y C/C++ quedan fuera (ver
[01-prd.md](01-prd.md), sección 5). Las menciones a "cuatro lenguajes" abajo
reflejan el alcance en el momento en que se investigó; se ajustan donde
cambian una conclusión, no en toda mención cosmética.
Todos los datos verificados contra documentación oficial en esta fecha.
Las capas gratuitas cambian sin aviso: **volver a verificar antes de comprometerse**
si se reabre esta decisión más adelante.

## 1. La pregunta y la respuesta corta

La pregunta era cómo tener IA en la plataforma sin invertir dinero, por ser un
proyecto escolar.

La respuesta corta es que **el proyecto probablemente no necesita ser gratuito,
porque no cuesta lo que parece**. La cifra de 1,000 generaciones al mes que
aparece en [03-costos.md](03-costos.md) es una proyección hipotética de escala,
no un requisito. El PRD pide que diez estudiantes ajenos al equipo generen al
menos una vez. Sumando calibración, desarrollo, prueba con usuarios y demo, el
semestre completo son alrededor de **600 generaciones**.

| Modelo | Semestre completo (~600 generaciones) | En pesos |
|---|---|---|
| Claude Haiku 4.5 | $8.40 | ~$155 |
| Claude Sonnet 5 | $14.40 | ~$266 |
| Claude Opus 5 | $40.80 | ~$755 |

Armar colas, manejo de errores 429, fallback entre proveedores y control de
ráfagas cuesta más horas de trabajo de las que valen $266 pesos. **La opción
gratuita solo tiene sentido si además no hay forma de conseguir esos $266, o si
el equipo quiere la experiencia de integrarse contra varios proveedores.**

Dicho eso, hay dos caminos gratuitos legítimos y uno de ellos es mejor que
pagar. Van en la sección 4.

## 2. El hallazgo que importa más que el proveedor

El criterio de validez del PRD es que 17 de 20 archivos generados corran sin
errores de sintaxis, import o nombre. **Esa es exactamente la clase de error que
un compilador detecta solo, gratis, en milisegundos.**

Pasar el archivo generado por un validador y, si falla, reenviarlo al modelo con
el mensaje de error pegado, cambia los resultados de forma dramática:

| Configuración | Primer intento | Con puerta + 1 reintento |
|---|---|---|
| API de frontera (Claude, Gemini) | ~18-19 / 20 | ~19-20 / 20 |
| Qwen3.6-35B local, cuantizado Q4 | ~14-17 / 20 | **~18-19 / 20** |
| Gemma 4 12B local, cuantizado Q4 | ~11-14 / 20 | ~16-18 / 20 |
| gpt-oss-20b | ~12-15 / 20 | ~16-18 / 20 |

*(Estimaciones: ningún benchmark público mide "generar un archivo de pruebas
ejecutable en varios frameworks con prosa en español".)*

**La fila de la puerta pesa más que la fila del modelo.** Un modelo abierto de
12B con validación llega más lejos que un modelo de frontera sin ella. Eso es lo
que vuelve viables las opciones baratas.

### 2.1 Choca con una decisión ya tomada

En el mapeo se decidió "no ejecutar nada" y se descartó explícitamente la opción
de validar sintaxis. Esa decisión se tomó evaluando seguridad, sin saber que la
validación era también la palanca de costo y de calidad.

**Validar sintaxis no es ejecutar.** `py_compile`, `tsc --noEmit`, `javac` y
`g++ -fsyntax-only` parsean sin correr nada: no hay proceso del usuario
ejecutándose, así que el riesgo que motivó la decisión original sigue en cero.

Vale la pena reabrir la decisión. Complicación práctica a considerar: en un
despliegue serverless (Vercel) no hay intérprete de Python disponible (para
JavaScript/TypeScript, el otro lenguaje del MVP, sí hay). Las salidas
posibles, de menor a mayor esfuerzo:

1. **Validar solo la sintaxis con `tree-sitter` compilado a WASM.** Existen
   gramáticas para los lenguajes del MVP, corre en JavaScript puro, funciona en
   serverless y tarda milisegundos. Detecta errores de sintaxis, **no** de
   import ni de nombre.
2. **Mover el despliegue a un servidor Node con los compiladores instalados.**
   Detecta las tres clases de error, pero implica administrar el servidor.
3. **Dejarlo fuera del MVP** y documentar por qué, aceptando la tasa de acierto
   sin puerta.

## 3. Restricción transversal: privacidad de los datos

El producto recibe **código que el estudiante escribió para su clase**. Buena
parte de las capas gratuitas se pagan con esos datos.

| Proveedor (capa gratuita) | ¿Entrena con tus datos? | Nota |
|---|---|---|
| Google Gemini | **Sí** | Además, "revisores humanos pueden leer, anotar y procesar tu entrada y salida" |
| OpenAI (tokens por compartir datos) | **Sí** | Es literalmente el precio del programa |
| OpenRouter, modelos `poolside/*` y `nvidia/*` | **Sí** | Por endpoint, no por plataforma |
| Mistral modo gratuito | Sí **por defecto** | Se apaga con un interruptor. Los modelos *Labs* y *Preview* no se pueden apagar |
| Groq | No | Prohibido contractualmente |
| Cloudflare Workers AI | No | Sin distinción entre plan gratuito y pagado |
| OpenRouter `z-ai/glm-5.2:free` | No | Además retención cero |
| Vertex AI y APIs de pago | No | Trato de tier pagado |

**Detalle específico para México.** Los términos de Gemini dan trato de tier
pagado al tier gratuito únicamente en el Espacio Económico Europeo, Suiza y
Reino Unido. México no está en esa excepción, así que aquí aplica el texto
completo: *"No envíes información sensible, confidencial o personal a los
Servicios No Pagados."*

Esto no vuelve inviables esas opciones — se resuelve avisando en la interfaz—
pero es una decisión consciente que conviene poder defender en la presentación,
no descubrir cuando alguien pregunte.

## 4. Opciones, en orden de recomendación

### 4.1 Créditos Google Cloud Education (recomendado)

**$100 por profesor + $50 por estudiante.** México elegible, y `uanl.mx`
califica: Google pide "una dirección de correo con el dominio de tu escuela",
sin exigir `.edu`. Cubre Vertex AI.

Por qué es la mejor opción y no solo la más barata:

- **Vertex AI sirve modelos Claude**, así que el equipo se queda con el stack ya
  documentado sin reescribir nada.
- El trato de datos de Vertex es el de tier pagado: **no entrena** con lo que se
  envía. Resuelve la sección 3 de un golpe.
- Con un equipo de cinco son $350 dólares, unas 14,000 generaciones con
  Sonnet 5. Catorce veces lo que el proyecto necesita.

**Restricción: solo un profesor puede aplicar.** Los estudiantes no pueden
solicitarlo por su cuenta. El trámite tarda ~15 días hábiles, así que hay que
arrancarlo en la primera semana del semestre o deja de servir.

### 4.2 Pagar de la bolsa del equipo

$155 a $266 pesos por el semestre completo, según el modelo. Cero fricción, cero
límites de ráfaga, cero problema de privacidad, cero trabajo de integración
extra. Es la opción que menos tiempo consume.

### 4.3 Gratuito de verdad, si las dos anteriores fallan

Los tres que sobreviven al filtro de privacidad:

| | OpenRouter | Cloudflare Workers AI | Groq |
|---|---|---|---|
| Modelo | `z-ai/glm-5.2:free` | `@cf/qwen/qwen3-30b-a3b-fp8` | `openai/gpt-oss-120b` |
| ¿Tarjeta? | No | No documentado | No |
| ¿Entrena? | No, retención cero | No | No |
| Capacidad | 50/día | ~126/día | ~41/día |
| Cuello de botella | 50/día sin comprar créditos | 10,000 neuronas/día | **8K tokens/min** |
| Streaming | Sí | Sí | Sí |
| Compatible OpenAI | Sí | Sí | Sí |

Notas de implementación:

- Los tres son compatibles con el SDK de OpenAI, así que cambiar de proveedor es
  cambiar una URL base. Conviene construirlo así desde el inicio.
- **OpenRouter**: mandar `provider: {"data_collection": "deny"}` en cada
  petición, para que un cambio de ruteo no tire la petición en un endpoint que sí
  entrena. Una compra única de $10 (de por vida, no mensual) sube el límite a
  1,000/día permanentemente.
- **Cloudflare**: `max_tokens` tiene default de **256**. Sin fijarlo en ~2,400,
  todos los archivos de prueba se cortan a media función. Dos cosas no se pueden
  verificar en la documentación y hay que probarlas: si el registro exige tarjeta
  y si `max_tokens: 2200` se acepta.
- **Groq**: el límite que muerde no es el de 30 peticiones/minuto que anuncia,
  sino el de 8,000 tokens/minuto: con ~4,800 tokens por generación son **1.6 por
  minuto**. Necesita cola del lado del cliente para que la ráfaga del día de
  entrega se convierta en espera y no en error.

### 4.4 Modelos locales

Costo marginal cero real, pero exige hardware. Lo verificado:

| Modelo | Tamaño en disco (Q4) | VRAM | Español |
|---|---|---|---|
| `gemma4:12b-it-qat` | 7.2 GB | 8 GB | **Mejor de su categoría**: 35+ idiomas soportados |
| `gpt-oss-20b` | — | 16 GB | Sin declaración multilingüe |
| `qwen3.8:27b` | 18 GB | 24 GB | 119 idiomas; MMMLU español 82.8 en el 32B |
| `qwen3.6:35b-a3b` | 21 GB | 24 GB | SWE-bench Verified 73.4 |

Sin GPU no es viable para uso interactivo: un modelo denso de 32B en CPU corre a
**3.54 tokens/segundo** (medido, no estimado), lo que da ~10 minutos por
generación. Sigue siendo una arquitectura legítima si se convierte en trabajo en
cola con aviso posterior, pero deja de ser una petición síncrona.

**El español es el riesgo, no el código.** La explicación pedagógica es el
producto, y los modelos especializados en código son los que peor escriben
prosa. Solo Gemma 4 y Nemotron declaran soporte de español explícitamente;
Ornith, KAT-Coder y gpt-oss no hacen ninguna afirmación multilingüe. El modo de
falla típico no lo detecta ningún benchmark: el modelo mete español en los
identificadores del código, o se pasa al inglés a media explicación.

**Mitigación que vale para cualquier modelo:** partir en dos llamadas. La
primera genera solo el archivo de pruebas, temperatura baja, instrucción de "solo
código". La segunda toma ese código y produce la explicación en español. Cuesta
~1.3× tokens y elimina la contaminación por completo. Además permite usar un
modelo distinto para cada parte.

## 5. Descartados, con la razón

| Opción | Por qué no |
|---|---|
| **Gemini tier gratuito** | Entrena con tus datos y revisores humanos leen el contenido. México fuera de la excepción europea |
| **GitHub Models** | **Retirado el 30 de julio de 2026.** El endpoint devuelve 410. Todo tutorial que lo recomiende está obsoleto |
| **Azure for Students** | Microsoft bloquea explícitamente los modelos Claude en cuentas estudiantiles: 0 RPM, 0 TPM. Azure OpenAI aparece como N/A en toda tabla de cuota estudiantil |
| **Crédito de $300 de Google Cloud** | Textual: "el crédito de $300 no puede pagar costos de Gemini API en AI Studio" |
| **Cerebras** | Ya no tiene capa gratuita: $5 a 30 días y exige tarjeta verificada |
| **Google Colab** | Su ToS **prohíbe** servir endpoints web, no es solo que sea poco práctico. Todo tutorial de "Ollama en Colab con ngrok" describe un patrón que puede costar la cuenta |
| **Kaggle** | 30 horas de GPU por semana, pero sin ningún mecanismo de endpoint persistente |
| **Hugging Face Spaces ZeroGPU** | Endpoint público real y hardware excelente, pero 5 minutos de GPU al día = ~8 generaciones diarias |
| **Oracle Cloud Always Free** | Servidor real y permanente, sin GPU: ~7-12 minutos por generación |
| **Programas de Anthropic** | No hay tier gratuito. Claude Campus cerró convocatoria, Claude for Education es contrato institucional, ninguna universidad mexicana es socia |
| **Programas estudiantiles de OpenAI** | Todos limitados a EE.UU. y Canadá, y los créditos son de ChatGPT/Codex, no de API |

Dos que **sí** sirven para experimentar aunque no para producción: Colab y
Kaggle son ideales para correr el conjunto de evaluación de la Fase 2 contra
varios modelos candidatos. Eso es exactamente para lo que están hechos.

## 6. Pista local sin verificar

La UANL lanzó **EsencIA** en agosto de 2025, una estrategia institucional de IA
cuyos aliados declarados incluyen Microsoft, AWS, Google y Meta, con un
"AI HUB-UANL" con laboratorios e incubación de proyectos. No hay información
pública sobre qué reciben los estudiantes.

Contacto: `contactouni@uanl.mx`. Cuesta un correo y podría resolver el problema
completo.

## 7. Decisión final: se cierra esta investigación

El equipo decidió pagar la API de Claude directamente: **$20 USD propios**,
Sonnet 5, cuota de 10 por usuario con sesión. Con eso el semestre completo
cuesta $14.40 (sección "Lo que realmente cuesta el proyecto" en
[03-costos.md](03-costos.md)) y sobran $5.60 de margen. Perseguir crédito
gratuito deja de tener sentido: el tiempo que cuesta el trámite de quince días
hábiles de Google Cloud Education vale más que los $266 pesos que se ahorraría.

Este documento **no fue trabajo perdido**. Queda como referencia si el
presupuesto se agota antes de tiempo, si el equipo crece, o si algún día se
necesita un plan de respaldo sin fricción de pago. La opción más rápida de
reactivar, si hiciera falta, es `z-ai/glm-5.2:free` de OpenRouter (sección 4.3):
no entrena con los datos, no pide tarjeta, y cambiar a ella es cambiar una
variable de entorno si la capa del modelo se construyó detrás de una interfaz
compatible con el SDK de OpenAI (recomendación que sigue vigente,
independientemente del proveedor final).

## 8. Decisiones

- [x] ¿Se reabre la decisión de validar sintaxis? Sí, y ya se resolvió: reglas
      por framework, sin parseo pesado. Ver
      [02-arquitectura.md](02-arquitectura.md) sección 2.11 y RF-20.
- [x] ¿Quién habla con el profesor para los créditos de Google Cloud Education?
      Nadie: el equipo paga directo, no hace falta el trámite.
- [x] ¿Alguien manda el correo a `contactouni@uanl.mx`? No es necesario para el
      MVP. Sigue siendo una pista barata si el presupuesto se acaba a mitad de
      semestre.
- [x] Aviso de entrenamiento en la interfaz. **Ya no aplica**: la API de pago de
      Claude no entrena con los datos enviados (verificado en el mapeo de
      modelos y precios). El aviso que sí sigue siendo necesario es otro,
      documentado en RF-12: que las pruebas generadas no fueron ejecutadas.

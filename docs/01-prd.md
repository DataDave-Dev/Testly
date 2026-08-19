# PRD — Testly (generador pedagógico de pruebas unitarias)

Fecha: 2026-08-19
Versión: 0.6

## 1. Problema

Escribir pruebas unitarias es de lo último que aprende un estudiante de
programación, y casi siempre lo aprende mal o no lo aprende. Se atora en tres
puntos concretos:

1. **No sabe qué probar.** Entiende la sintaxis de `assert` pero no se le ocurre
   qué casos valen la pena. Escribe una prueba del caso feliz y da por terminado.
2. **No conoce el framework.** Sabe Python pero nunca vio un `fixture` de pytest,
   ni un `@ParameterizedTest` de JUnit, ni cómo se compila un `TEST()` de
   GoogleTest.
3. **No ve para qué sirve.** Como sus programas de clase son de 50 líneas y él
   los probó a mano, probar le parece burocracia.

Las alternativas fallan cada una a su modo. El profesor enseña el framework pero
no alcanza a revisar caso por caso con cada alumno. Los tutoriales usan siempre
el mismo ejemplo de calculadora, que no se parece al código del estudiante. Y un
asistente de IA generalista le entrega el archivo de pruebas terminado sin
explicar el criterio, con lo que el estudiante queda igual de incapaz de escribir
el siguiente por su cuenta.

## 2. Objetivos

### Objetivo general

Desarrollar una plataforma web que reciba un fragmento de código, genere pruebas
unitarias para él mediante un modelo de lenguaje, y explique el criterio de
selección de cada caso, de modo que el estudiante desarrolle la capacidad de
escribir sus propias pruebas.

### Objetivos específicos

1. Permitir el envío de código en Python y JavaScript/TypeScript. Java y
   C/C++ quedan fuera del MVP (ver sección 5).
2. Generar un archivo de pruebas listo para copiar, en el framework
   convencional de cada lenguaje.
3. Acompañar cada caso de prueba con la explicación de qué cubre y por qué se
   eligió.
4. Permitir copiar o descargar el archivo generado.
5. Implementar autenticación y persistencia del historial por usuario.
6. Controlar el costo operativo mediante límites de uso por usuario.
7. Documentar arquitectura, decisiones técnicas y resultados.

## 3. Justificación

El proyecto es viable en el alcance de un semestre porque el componente difícil
—generar pruebas idiomáticas en los lenguajes del MVP (Python y
JavaScript/TypeScript) y sus frameworks— se resuelve delegando en un modelo de
lenguaje existente en lugar de construir generadores propios. Eso deja el esfuerzo del equipo en lo que sí es construible: el
producto, la calidad pedagógica de la explicación y el control de costos.

Técnicamente cubre desarrollo web full-stack, autenticación, persistencia,
integración con una API externa de pago y diseño de interfaz. Todo evaluable.

Como valor agregado, el tema del proyecto es la práctica de ingeniería que más
se descuida en la formación: el equipo aprende de pruebas mientras construye.

## 4. Usuarios

**Usuario principal:** estudiante de programación de nivel introductorio a
intermedio. Sabe escribir funciones pero no sabe probarlas, y no conoce el
framework de pruebas de su lenguaje. Usa la plataforma desde laptop,
ocasionalmente desde celular.

**Usuario secundario (fuera del MVP):** profesor que quiere material de apoyo
para enseñar pruebas con el código real de sus grupos.

## 5. Alcance

### Dentro del MVP

- Landing pública: qué hace la plataforma, un poco de teoría de pruebas
  unitarias, ejemplos, llamada a probarlo
- Generación **sin necesidad de cuenta**, con cuota reducida por IP
- Cuenta opcional (sin contraseña, magic link) para subir la cuota y guardar
  historial
- Editor donde pegar código, con selección de lenguaje y de framework
- Generación del archivo de pruebas
- Explicación por caso: qué cubre y por qué se eligió
- Validación por reglas de framework antes de mostrar el resultado como válido
- Detección heurística de pruebas tautológicas (assert True, aserciones triviales, mocks amañados) antes de mostrar el resultado
- Copiar y descargar el archivo generado
- Historial personal, consultable y re-abrible (solo con cuenta)
- Límite de generaciones por periodo, distinto para anónimo y autenticado
- Manejo de errores visible al usuario (falla de API, código demasiado largo)
- Aviso explícito de que las pruebas no fueron ejecutadas

### Fuera del MVP

- Soporte de Java y C/C++ (JUnit 5, GoogleTest/Catch2). El MVP cubre Python y
  JavaScript/TypeScript; los otros dos quedan como trabajo futuro, ver
  [04-plan.md](04-plan.md)
- Ejecución de las pruebas generadas
- Reporte de cobertura
- Detección de errores o análisis de estilo del código de entrada
- Cuentas de profesor, grupos o reportes agregados
- Chat de seguimiento sobre un resultado
- Pruebas de integración o de extremo a extremo
- Análisis de repositorios completos o de más de un archivo
- Generación de mocks para dependencias externas complejas (red, base de datos)
- Aplicación móvil nativa

## 6. Frameworks por lenguaje

| Lenguaje | Por defecto | Alternativa |
|---|---|---|
| Python | pytest | unittest |
| JavaScript / TypeScript | Vitest | Jest |

El sistema propone el que está por defecto y el usuario puede cambiarlo. Las
alternativas son un valor más en un selector y una línea más en el prompt: no
justifican dejarlas fuera.

**Java (JUnit 5) y C/C++ (GoogleTest, Catch2) quedan fuera del MVP** (ver
sección 5). Si se agregan más adelante, es la misma tabla con dos filas más.

## 7. Requisitos funcionales

| ID | Requisito |
|---|---|
| RF-01 | El sistema permite crear una sesión con correo, sin contraseña (magic link) |
| RF-02 | El sistema permite cerrar sesión |
| RF-03 | Cualquier visitante, con o sin sesión, puede pegar código y enviarlo |
| RF-04 | El usuario puede seleccionar el lenguaje; el sistema propone uno por defecto |
| RF-05 | El usuario puede seleccionar el framework; el sistema propone el de la tabla de la sección 6 |
| RF-06 | El sistema rechaza envíos que excedan el tamaño máximo, con mensaje claro |
| RF-07 | El sistema devuelve un archivo de pruebas ejecutable en el framework elegido |
| RF-08 | Cada caso de prueba va acompañado de la explicación de qué cubre y por qué |
| RF-09 | El sistema indica cuando el código recibido no es apto para pruebas unitarias, en lugar de generar pruebas artificiales |
| RF-10 | El usuario puede copiar el archivo de pruebas al portapapeles |
| RF-11 | El usuario puede descargar el archivo con el nombre convencional del framework |
| RF-12 | La interfaz advierte que las pruebas no fueron ejecutadas y deben verificarse |
| RF-13 | Si hay sesión, cada generación se guarda asociada al usuario que la generó. Sin sesión, no se guarda nada |
| RF-14 | El usuario con sesión puede ver la lista de sus generaciones previas, ordenada por fecha |
| RF-15 | El usuario con sesión puede abrir una generación previa y ver el código, las pruebas y la explicación |
| RF-16 | El usuario con sesión puede eliminar una generación de su historial |
| RF-17 | El sistema aplica un límite de generaciones por periodo: **3 por día por IP** sin sesión, uno mayor y configurable con sesión |
| RF-18 | El sistema muestra cuántas generaciones quedan en el periodo, con o sin sesión |
| RF-19 | Si la API del modelo falla, el sistema informa al usuario y no cobra el intento contra su límite |
| RF-20 | El sistema valida contra reglas por framework (imports correctos, sin mezclar versiones) antes de mostrar el resultado como válido; si falla, lo marca visualmente sin bloquear la descarga |
| RF-21 | La landing explica qué hace la plataforma y qué son las pruebas unitarias, con al menos un ejemplo, antes de llevar al editor |
| RF-22 | El flujo sin sesión pasa por un desafío anti-bot (Cloudflare Turnstile) antes de generar |
| RF-23 | El usuario con sesión puede eliminar su cuenta. La eliminación es un **soft delete**: no se borra físicamente, se marca como eliminada y el usuario deja de poder iniciar sesión con ella. Ver [02-arquitectura.md](02-arquitectura.md), sección 2.13, y [08-limites.md](08-limites.md) |
| RF-24 | El sistema aplica una heurística estática que detecta patrones típicos de pruebas tautológicas (`assert True`, aserciones sobre constantes, mocks que retornan justo lo que la prueba afirma) y marca visualmente la prueba como sospechosa, sin bloquear la descarga. No requiere ejecutar código — mismo criterio de costo y seguridad que RF-20 |

## 8. Requisitos no funcionales

| ID | Requisito |
|---|---|
| RNF-01 | El resultado se entrega en menos de 45 segundos en el 90% de los casos |
| RNF-02 | La respuesta se muestra en streaming: el usuario ve texto antes de que termine |
| RNF-03 | El código del usuario nunca se ejecuta, ni las pruebas generadas |
| RNF-04 | Las credenciales de la API del modelo viven solo en el servidor, nunca en el cliente |
| RNF-05 | No hay contraseñas que almacenar (sesión sin contraseña); el enlace de acceso expira y es de un solo uso |
| RNF-06 | La salida del modelo se renderiza de forma segura, sin inyección de HTML |
| RNF-07 | La interfaz es usable en pantallas desde 375px de ancho |
| RNF-08 | El costo por generación se registra por cada llamada, para poder auditar el gasto |

RNF-01 sube de 30 a 45 segundos respecto de la versión anterior: generar un
archivo de pruebas es una respuesta considerablemente más larga que explicar
código. Esto endurece el requisito de duración de función del hosting; ver
[02-arquitectura.md](02-arquitectura.md), sección 2.8.

## 9. Criterios de éxito

| Criterio | Meta |
|---|---|
| Funcionalidad | Los 23 requisitos funcionales operan de extremo a extremo |
| Cobertura de casos | Sobre el conjunto de evaluación de 20 funciones con casos documentados, las pruebas generadas cubren el caso feliz, el borde y el de error en al menos 15 |
| Validez | Al ejecutar a mano las pruebas de esas 20, al menos 17 corren sin errores de sintaxis, import o nombre |
| Pruebas vacías | Cero pruebas tautológicas (que pasan sin importar la implementación) en la muestra evaluada. Primera barrera: heurística RF-24 |
| Costo | El costo promedio por generación se mantiene dentro del presupuesto de [03-costos.md](03-costos.md) |
| Uso real | Al menos 10 estudiantes ajenos al equipo generan al menos una vez |
| Aprendizaje percibido | En encuesta breve, la mayoría reporta entender mejor qué casos vale la pena probar |

El criterio de validez se mide **a mano** en la Fase 2, corriendo las pruebas
generadas en local. Que el producto no ejecute pruebas no significa que el
equipo no deba ejecutarlas para evaluarlo: es exactamente la medición que dice
si el producto sirve.

## 10. Postura sobre uso indebido

Hay que decirlo sin rodeos: **si la tarea asignada es escribir pruebas
unitarias, esta herramienta puede hacerla.** Es una tensión real del proyecto,
no un detalle a minimizar, y la postura de diseño es la siguiente:

- La explicación del criterio de cada caso no es opcional ni se puede omitir de
  la salida. El estudiante que solo quiere el archivo tiene que pasar por ella.
- El prompt instruye al modelo a explicar el razonamiento —qué se rompe en cada
  caso, por qué ese borde y no otro— en lugar de solo etiquetar cada prueba.
- El código pegado se guarda en el historial junto con lo generado. Un profesor
  que sospeche puede pedirle al estudiante que explique el criterio.
- La plataforma no acepta enunciados de problemas para generar código y luego
  probarlo: solo acepta código ya escrito.

Esto no es infalible y no se presenta como tal. La postura honesta es que la
herramienta enseña mejor de lo que un estudiante aprende copiando de un
asistente generalista, que es la alternativa que ya tiene disponible.

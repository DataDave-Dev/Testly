## Purpose

Define la estructura y composición del conjunto de evaluación de Testly: el
conjunto fijo de 20 funciones (Python + JavaScript/TypeScript), con sus casos
de prueba documentados, que la Fase 2 (Calibración) usa para medir `effort`
sobre Sonnet 5 y verificar los criterios de éxito de
[01-prd.md](../../../docs/producto/01-prd.md) sección 9 (cobertura, validez,
cero pruebas tautológicas).

## Requirements

### Requirement: Tamaño y composición del conjunto
El conjunto de evaluación SHALL estar compuesto por exactamente 20 funciones
en total. De esas 20, entre 3 y 4 SHALL ser fragmentos "difíciles de probar"
(Requirement: Composición de fragmentos difíciles de probar); el resto SHALL
ser funciones "normales" con casos documentados completos (camino feliz,
valor límite, entrada inválida).

#### Scenario: Conteo total del conjunto
- **WHEN** se cuentan todos los archivos de función documentados bajo
  `docs/evaluacion/`
- **THEN** el total es 20, sin funciones adicionales fuera de ese conteo

#### Scenario: Los difíciles cuentan dentro del total
- **WHEN** se identifican los fragmentos "difíciles de probar" dentro del
  conjunto
- **THEN** están incluidos en las 20 funciones totales, no agregados aparte

### Requirement: Reparto por lenguaje
El conjunto SHALL incluir 10 funciones en Python y 10 funciones en
JavaScript/TypeScript.

#### Scenario: Verificación del reparto
- **WHEN** se agrupan las 20 funciones por lenguaje
- **THEN** hay exactamente 10 marcadas `python` y exactamente 10 marcadas
  `javascript` o `typescript`

### Requirement: Manifest estructurado de casos por función
Cada función del conjunto SHALL tener un manifest en formato YAML o JSON que
documente su lista de casos de prueba esperados. Cada caso SHALL incluir como
mínimo: `tipo` (uno de `camino_feliz`, `valor_limite`, `entrada_invalida`),
`entrada` y `esperado` (salida o comportamiento esperado explícito). El
manifest NO SHALL depender de prosa libre como único medio de describir un
caso.

#### Scenario: Caso de camino feliz documentado
- **WHEN** se lee el manifest de una función normal del conjunto
- **THEN** existe al menos un caso con `tipo: camino_feliz`, con `entrada` y
  `esperado` explícitos

#### Scenario: Caso de valor límite documentado
- **WHEN** se lee el manifest de una función normal del conjunto
- **THEN** existe al menos un caso con `tipo: valor_limite`, con `entrada` y
  `esperado` explícitos

#### Scenario: Caso de entrada inválida documentado
- **WHEN** se lee el manifest de una función normal del conjunto
- **THEN** existe al menos un caso con `tipo: entrada_invalida`, con
  `entrada` y `esperado` explícitos (por ejemplo, la excepción o el código de
  error que debe producirse)

### Requirement: Matriz de atributos de variedad de forma
El conjunto de funciones normales SHALL cubrir, en conjunto, la siguiente
matriz de atributos: al menos una función que retorna un valor escalar, al
menos una que retorna una colección, al menos una que puede retornar `None`
o equivalente; al menos una que señala error lanzando una excepción y al
menos una que señala error retornando un código/valor de error; al menos una
función con parámetros de valor por defecto; y al menos una función
recursiva.

#### Scenario: Verificación de la matriz de atributos
- **WHEN** se revisan los metadatos de atributos de las funciones normales
  del conjunto
- **THEN** cada atributo de la matriz (tipo de retorno escalar/colección/
  `None`, excepción vs. código de error, parámetro con default, recursión)
  está cubierto por al menos una función

### Requirement: Composición de fragmentos difíciles de probar
El conjunto SHALL incluir los siguientes fragmentos difíciles de probar,
cada uno documentado con el comportamiento esperado del sistema (RF-09: "esto
no es apto para pruebas unitarias") en vez de una lista de casos de prueba
convencional:
- Un script sin funciones (código a nivel de módulo, sin ninguna función
  definida).
- Una función cuyo resultado depende de la hora del sistema.
- Una función que lee un archivo del sistema de archivos.
- Una función con una dependencia de red o de base de datos.

#### Scenario: Cada fragmento difícil está presente
- **WHEN** se listan los fragmentos difíciles del conjunto
- **THEN** están presentes los cuatro tipos: script sin funciones, dependencia
  de la hora del sistema, lectura de archivo, y dependencia de red/base de
  datos

#### Scenario: Un fragmento difícil no trae lista de casos convencional
- **WHEN** se lee el manifest de un fragmento difícil
- **THEN** en vez de casos `camino_feliz`/`valor_limite`/`entrada_invalida`,
  documenta el comportamiento esperado del sistema ante código no apto para
  pruebas unitarias (RF-09)

### Requirement: Ubicación del conjunto en el repositorio
El conjunto de evaluación completo (funciones, manifests de casos y matriz de
atributos) SHALL vivir bajo `docs/evaluacion/`.

#### Scenario: Ubicación verificable
- **WHEN** se busca el conjunto de evaluación en el repositorio
- **THEN** todos sus archivos están contenidos dentro de `docs/evaluacion/`

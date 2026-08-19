# Matriz de atributos — conjunto de evaluación

Fecha: 2026-08-19

Atributos fijados en
[design.md](../../openspec/changes/armar-conjunto-evaluacion/design.md#3-manifest-estructurado-yamljson-en-vez-de-tabla-en-prosa)
(Decisión 4) que las 20 funciones normales deben cubrir en conjunto, dentro
de cada subconjunto de lenguaje, para que "borde" e "inválido" no signifiquen
lo mismo en las 20 funciones.

## Python (`normales/python/`)

| Función | retorno_escalar | retorno_coleccion | retorno_none | lanza_excepcion | codigo_error | parametro_default | recursion |
|---|---|---|---|---|---|---|---|
| `promedio` | ✅ | | | ✅ (invalida) | | | |
| `numeros_pares` | | ✅ | | ✅ (invalida) | | | |
| `buscar_usuario` | | | ✅ | ✅ (invalida) | | | |
| `dividir` | ✅ | | | ✅ | | | |
| `validar_contrasena` | | | | | ✅ | | |
| `formatear_precio` | | | | ✅ (invalida) | | ✅ | |
| `factorial` | ✅ | | | ✅ (invalida) | | | ✅ |
| `es_palindromo` | ✅ | | | ✅ (invalida) | | | |

Cobertura Python: los 7 atributos presentes al menos una vez ✅.

## JavaScript/TypeScript (`normales/javascript/`)

| Función | retorno_escalar | retorno_coleccion | retorno_none | lanza_excepcion | codigo_error | parametro_default | recursion |
|---|---|---|---|---|---|---|---|
| `promedio` | ✅ | | | ✅ (invalida) | | | |
| `numerosPares` | | ✅ | | ✅ (invalida) | | | |
| `buscarUsuario` | | | ✅ | ✅ (invalida) | | | |
| `dividir` | ✅ | | | ✅ | | | |
| `validarContrasena` | | | | | ✅ | | |
| `formatearPrecio` | | | | ✅ (invalida) | | ✅ | |
| `factorial` | ✅ | | | ✅ (invalida) | | | ✅ |
| `esPalindromo` | ✅ | | | ✅ (invalida) | | | |

Cobertura JS/TS: los 7 atributos presentes al menos una vez ✅.

## Nota sobre solapamiento

Varios atributos ocurren juntos en una misma función (ej. `promedio` retorna
escalar en el camino feliz y lanza excepción en el caso inválido). Eso es
intencional: el manifest requiere de todos modos los tres tipos de caso
(camino_feliz, valor_limite, entrada_invalida) por función, así que el "cómo
falla" (excepción vs. código de error) casi siempre acompaña a algún otro
atributo estructural (tipo de retorno, default, recursión). La matriz exige
que cada atributo esté cubierto **al menos una vez**, no que cada función
cubra exactamente uno.

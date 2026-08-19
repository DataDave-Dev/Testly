## 1. Preparación

- [x] 1.1 Crear `docs/evaluacion/` con subcarpetas `normales/python/`,
      `normales/javascript/` y `dificiles/`
- [x] 1.2 Definir la plantilla YAML del manifest de casos
      (`docs/evaluacion/_plantilla-manifest.yaml`), con los campos `tipo`
      (`camino_feliz` | `valor_limite` | `entrada_invalida`), `entrada` y
      `esperado`
- [x] 1.3 Crear `docs/evaluacion/matriz-atributos.md` con la matriz fijada en
      `design.md` (retorno escalar / colección / `None`, excepción vs.
      código de error, parámetro con default, recursión) y una tabla vacía
      para marcar qué función cubre qué atributo, a llenar en las secciones
      2 y 3

## 2. Funciones normales — Python (8)

Cada tarea produce el código de la función + su manifest de casos
(camino feliz, valor límite, entrada inválida) bajo
`docs/evaluacion/normales/python/`.

- [x] 2.1 Función que retorna un valor escalar — `promedio.yaml`
- [x] 2.2 Función que retorna una colección (lista, dict, etc.) —
      `numeros_pares.yaml`
- [x] 2.3 Función que puede retornar `None` en algún camino —
      `buscar_usuario.yaml`
- [x] 2.4 Función que señala error lanzando una excepción ante entrada
      inválida — `dividir.yaml`
- [x] 2.5 Función que señala error retornando un código/valor de error ante
      entrada inválida (sin lanzar excepción) — `validar_contrasena.yaml`
- [x] 2.6 Función con al menos un parámetro de valor por defecto —
      `formatear_precio.yaml`
- [x] 2.7 Función recursiva — `factorial.yaml`
- [x] 2.8 Función adicional de variedad libre (combinación de atributos no
      repetida entre las 2.1-2.7) — `es_palindromo.yaml`

## 3. Funciones normales — JavaScript/TypeScript (8)

Mismo criterio que la sección 2, bajo
`docs/evaluacion/normales/javascript/`. Repetir la cobertura de la matriz de
atributos también dentro del subconjunto JS/TS, no solo una vez en Python.

- [x] 3.1 Función que retorna un valor escalar — `promedio.yaml`
- [x] 3.2 Función que retorna una colección (array, objeto, etc.) —
      `numerosPares.yaml`
- [x] 3.3 Función que puede retornar `undefined`/`null` en algún camino —
      `buscarUsuario.yaml`
- [x] 3.4 Función que señala error lanzando una excepción ante entrada
      inválida — `dividir.yaml`
- [x] 3.5 Función que señala error retornando un código/valor de error ante
      entrada inválida (sin lanzar excepción) — `validarContrasena.yaml`
- [x] 3.6 Función con al menos un parámetro de valor por defecto —
      `formatearPrecio.yaml`
- [x] 3.7 Función recursiva — `factorial.yaml`
- [x] 3.8 Función adicional de variedad libre (combinación de atributos no
      repetida entre las 3.1-3.7) — `esPalindromo.yaml`

## 4. Fragmentos difíciles de probar (4)

- [x] 4.0 Confirmado con el usuario: 2 en Python (script sin funciones,
      lectura de archivo), 2 en JavaScript/TypeScript (hora del sistema,
      red/DB) — conserva el 10/10 de la sección "Reparto por lenguaje"
- [x] 4.1 Script sin funciones (código a nivel de módulo, sin funciones
      definidas) — `script_sin_funciones.yaml`
- [x] 4.2 Función cuyo resultado depende de la hora del sistema —
      `depende_hora_sistema.yaml`
- [x] 4.3 Función que lee un archivo del sistema de archivos —
      `lee_archivo.yaml`
- [x] 4.4 Función con dependencia de red o base de datos —
      `dependencia_red_db.yaml`

## 5. Verificación y cierre

- [x] 5.1 Verificar el conteo total: 20 funciones, reparto 10 Python / 10
      JavaScript-TypeScript (incluye los 4 difíciles dentro del conteo) —
      verificado por conteo de archivos, ver transcripción de la sesión
- [x] 5.2 Verificar en `matriz-atributos.md` que los 7 atributos de la
      matriz están cubiertos al menos una vez en cada subconjunto de
      lenguaje — cobertura completa en ambos subconjuntos
- [x] 5.3 Corregir `docs/sprints/sprint-01.md`: fusionar los dos checkboxes
      ambiguos ("20 funciones" y "3-4 difíciles") en uno solo que refleje
      "20 en total, difíciles incluidos", y marcarlo como hecho
- [x] 5.4 Revisión por al menos otra persona del equipo (Definition of Done,
      [07-metodologia.md](../../../docs/gestion/07-metodologia.md) sección 6)
      — revisado por el usuario

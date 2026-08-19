## Descripción

<!-- Qué cambia y por qué. -->

## Tipo de cambio

- [ ] feat — funcionalidad nueva
- [ ] fix — corrección de bug
- [ ] refactor — cambio interno sin alterar comportamiento
- [ ] docs — documentación
- [ ] test — pruebas
- [ ] chore — mantenimiento, dependencias, config
- [ ] perf — rendimiento
- [ ] ci — pipeline / automatización

## Referencia

<!-- Obligatorio. RF del PRD (ej. RF-07), sprint o issue que resuelve. -->

## Capturas / evidencia visual

<!-- Antes/después o video si hay cambio visual. Escribe N/A si no aplica. -->

## Checklist de calidad

- [ ] Código legible y bien nombrado
- [ ] Funciones enfocadas (<50 líneas)
- [ ] Archivos cohesivos (<800 líneas)
- [ ] Sin anidación profunda (>4 niveles)
- [ ] Errores manejados explícitamente
- [ ] Sin `console.log` ni código de debug
- [ ] Sin código muerto o comentado

## Checklist de seguridad

- [ ] Sin secrets ni API keys hardcodeados
- [ ] Input de usuario validado
- [ ] Sin riesgo de XSS (salida del modelo sanitizada, ver RNF-06)
- [ ] Si toca auth, DB o API externa: revisado con cuidado extra

## Testing

- [ ] Pruebas unitarias agregadas o actualizadas
- [ ] Cobertura ≥80% en lo tocado
- [ ] Probado manualmente el flujo afectado
- [ ] Sin romper flujos existentes

## Pre-review

- [ ] CI / checks pasan
- [ ] Sin conflictos de merge
- [ ] Rama actualizada con `main`

## Notas para el reviewer

<!-- Contexto extra, decisiones tomadas, dudas puntuales. -->

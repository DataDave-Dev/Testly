## Purpose

Define los requisitos del scaffold inicial de Testly: proyecto Astro en la
raíz del repositorio, adapter de Cloudflare configurado en modo SSR, y build
y arranque local verificados. Es la base sobre la que corre el resto del
Sprint 2 (variables de entorno, modo mantenimiento, endpoint `/api/generar`,
pantalla mínima) — ver
[02-arquitectura.md](../../../docs/arquitectura/02-arquitectura.md).

## Requirements

### Requirement: Ubicación del proyecto Astro
El proyecto Astro SHALL inicializarse en la raíz del repositorio, sin
estructura de monorepo ni workspaces.

#### Scenario: package.json en la raíz
- **WHEN** se inspecciona la raíz del repositorio
- **THEN** existe `package.json` ahí, no dentro de una subcarpeta como
  `app/` o `web/`

#### Scenario: Sin configuración de workspaces
- **WHEN** se revisa `package.json`
- **THEN** no declara el campo `workspaces`

### Requirement: TypeScript en modo estricto
El proyecto SHALL usar TypeScript configurado en modo estricto (extendiendo
el preset estricto de Astro).

#### Scenario: tsconfig extiende el preset estricto
- **WHEN** se lee `tsconfig.json`
- **THEN** su campo `extends` referencia el preset `strict` (o superior) de
  `astro/tsconfigs/`

### Requirement: Adapter de Cloudflare en modo SSR
El proyecto SHALL tener el paquete `@astrojs/cloudflare` instalado como
dependencia y configurado como adapter en `astro.config.mjs` (o `.ts`), con
`output: 'server'`.

#### Scenario: Dependencia instalada
- **WHEN** se revisa `package.json`
- **THEN** `@astrojs/cloudflare` aparece en `dependencies`

#### Scenario: output en modo server
- **WHEN** se lee `astro.config.mjs`/`.ts`
- **THEN** `output` está fijado a `'server'` y `adapter` invoca la función
  del paquete `@astrojs/cloudflare`

### Requirement: Configuración de Wrangler para Cloudflare Workers
El proyecto SHALL incluir un archivo de configuración de Wrangler
(`wrangler.jsonc` o `wrangler.toml`) válido, apuntando al runtime de
Cloudflare **Workers** (no Pages).

#### Scenario: Archivo de configuración presente
- **WHEN** se inspecciona la raíz del repositorio
- **THEN** existe `wrangler.jsonc` o `wrangler.toml`

#### Scenario: compatibility_date fijado
- **WHEN** se lee el archivo de configuración de Wrangler
- **THEN** declara un `compatibility_date` explícito (no vacío ni ausente)

### Requirement: Build de producción sin errores
El comando de build del proyecto SHALL completar sin errores y producir un
output compatible con el runtime de Cloudflare Workers.

#### Scenario: pnpm run build exitoso
- **WHEN** se ejecuta `pnpm run build` desde la raíz del repositorio
- **THEN** el proceso termina con código de salida 0 y genera el directorio
  de salida del adapter de Cloudflare (`dist/`)

### Requirement: Arranque en modo desarrollo
El proyecto SHALL arrancar en modo desarrollo local sin errores.

#### Scenario: pnpm run dev arranca sin errores
- **WHEN** se ejecuta `pnpm run dev` desde la raíz del repositorio
- **THEN** el servidor de desarrollo de Astro arranca y sirve la página por
  defecto sin excepciones no controladas en consola

### Requirement: .gitignore cubre los artefactos del scaffold
El `.gitignore` del repositorio SHALL excluir los artefactos generados por
el proyecto Astro y por Wrangler.

#### Scenario: Artefactos de build y dependencias ignorados
- **WHEN** se lee `.gitignore` después de esta change
- **THEN** incluye entradas para `node_modules/`, `dist/`, `.astro/` y
  `.wrangler/`

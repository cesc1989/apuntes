# ADR-003 - Rubocop y Herb en el CI

A la vez para mis PRs a mano o para el código que escriba algún agente LLM. Ya es hora de tener estos dos linters configurados en los proyectos de DevAsPros. Empezando por los cinco que están en progreso y Puntapie.

Hacer esto viene de la mano de que ya voy aceptando que el desarrollo agentico es lo que se viene en la carrera. Entonces debo entender y saber cómo configurar estas herramientas para correr en local y en el CI. El objetivo final es tener más control sobre el código que genera el agente LLM.

Va de la mano junto con los skills propios. Es una forma de hacer un poco más determinista lo que hace el agente LLM y tener más certeza de que lo que hizo cumple con mis expectativas.

Las configuraciones a continuación fueron primero hechas y probadas en Cash Flow. De ahí repliqué a los demás.

## Rubocop

> [!Note]
> Solo corre sobre archivos .rb que no estén excluidos. Los archivos de las vistas quedan a cargo de Herb.

### Configuración

Esta es la configuración final que dejé para Cash Flow y que repliqué en los demás proyectos:
```yaml
inherit_from: .rubocop_todo.yml

plugins:
  - rubocop-rails

AllCops:
  TargetRubyVersion: 3.4
  Exclude:
    - "db/**/*"
    - "config/**/*"
    - "app/views/**/*"
    - "lib/tasks/*"
    - "spec/spec_helper.rb"
    - "spec/rails_helper.rb"
    - test_models.rb
    - test_rack_attack.rb
    - "bin/*"
    - vendor/**/*
  NewCops: disable # see https://docs.rubocop.org/rubocop/versioning.html#pending-cops

Style/Documentation:
  Enabled: false

Style/FrozenStringLiteralComment:
  Enabled: false

Style/SymbolArray:
  Exclude:
    - spec/factories/*

Style/StringLiterals:
  EnforcedStyle: double_quotes

Layout/LineLength:
  Max: 130

Metrics/AbcSize:
  Enabled: false

Metrics/MethodLength:
  Enabled: false

Metrics/ClassLength:
  Enabled: false

Metrics/BlockLength:
  Enabled: false

Metrics/CyclomaticComplexity:
  Enabled: false

Metrics/PerceivedComplexity:
  Enabled: false

Lint/AmbiguousOperatorPrecedence:
  Enabled: true

Lint/DuplicateBranch:
  Enabled: true

Lint/LiteralAssignmentInCondition:
  Enabled: true

Lint/UnusedPrivateMethod:
  Enabled: true

Lint/UselessRescue:
  Enabled: true

Rails/ActionControllerFlashBeforeRender:
  Enabled: true

Rails/ActionOrder:
  Enabled: true

Rails/CompactBlank:
  Enabled: true

Rails/DuplicateAssociation:
  Enabled: true

Rails/DurationArithmetic:
  Enabled: true

Rails/EnvLocal:
  Enabled: true

Rails/ExpandedDateRange:
  Enabled: true

Rails/FreezeTime:
  Enabled: true

Rails/NegateInclude:
  Enabled: true

Rails/Pluck:
  Enabled: true

Rails/RedundantActiveRecordAllMethod:
  Enabled: true

Rails/RedundantPresenceValidationOnBelongsTo:
  Enabled: true

Rails/ResponseParsedBody:
  Enabled: true

Rails/SelectMap:
  Enabled: true

Rails/TransactionExitStatement:
  Enabled: true

Rails/WhereExists:
  Enabled: true

Rails/WhereMissing:
  Enabled: true

Rails/WhereNot:
  Enabled: true

Rails/WhereRange:
  Enabled: true

Style/ArrayFirstLast:
  Enabled: true

Style/ComparableBetween:
  Enabled: true

Style/ComparableClamp:
  Enabled: true

Style/HashExcept:
  Enabled: true

Style/HashSlice:
  Enabled: true

Style/MapIntoArray:
  Enabled: true

Style/RedundantFilterChain:
  Enabled: true

Style/ReturnNilInPredicateMethodDefinition:
  Enabled: true

Style/SafeNavigationChainLength:
  Enabled: true

Style/StaticClass:
  Enabled: true

Style/YodaExpression:
  Enabled: true

```

Son reglas que creo son necesarias, lógicas o saludables para el proyecto en que estén verificandose.

### En el CI

En el archivo de actions se configuró así:
```yaml
rubocop:
	name: RuboCop
	runs-on: ubuntu-24.04
	steps:
		- uses: actions/checkout@v7
		- uses: ruby/setup-ruby@v1
			with:
				ruby-version: 3.4.8
				bundler-cache: true
		- name: Run RuboCop
			run: bundle exec rubocop --format github
```

La bandera `format github` es para que [la salida sea más amigable con Github Action](https://docs.rubocop.org/rubocop/latest/formatters.html#github-actions-formatter).

## Herb

> [!Note]
> Para los archivos de las vistas. Los archivos .erb.

Con Herb podemos determinar errores en los archivos ERB de manera precisa y confiable. Tiene varios niveles para los reportes. Inicialmente elegí solo `errors` pero corregí todo para tener un inicio más limpio.

### Configuración y Ejecución

```yaml
# This file configures Herb for your project and team.
# Settings here take precedence over individual editor preferences.
#
# Herb is a suite of tools for HTML+ERB templates including:
# - Linter: Validates templates and enforces best practices
# - Formatter: Auto-formats templates with intelligent indentation
# - Language Server: Provides IDE support (VS Code, Zed, Neovim, etc.)
#
# Website: https://herb-tools.dev
# Configuration: https://herb-tools.dev/configuration
# GitHub Repo: https://github.com/marcoroth/herb
#

version: 0.11.2

# Framework and template engine configuration
#
# Options: ruby | actionview | hanami | sinatra (default: ruby)
framework: actionview

files:
  # Reportes y archivos generados que no son plantillas del proyecto
  exclude:
    - 'docs/**/*'
    - 'spec/coverage/**/*'
    - 'coverage/**/*'
    - 'public/**/*'
    - 'tmp/**/*'
    - 'vendor/**/*'
    - 'node_modules/**/*'


linter:
  enabled: true


formatter:
  enabled: false
  indentWidth: 2
  indentStyle: space
  maxLineLength: 80

```

Cuando se corre el comando:
```bash
npx --yes @herb-tools/linter@0.11.2
```

La salida incluye un resumen como este:
```bash
Summary:
  Checked      55 files
  Files        18 with offenses | 37 clean (55 total)
  Failing      0 offenses
  Not failing  11 warnings | 13 info | 2 hints (26 offenses across 18 files, below --fail-level=error)
  Fixable      26 offenses | 1 autocorrectable using `--fix` | 3 more using `--fix-unsafely`
  Rules        149 enabled | 22 not enabled
  Start at     14:03:28
  Duration     3479ms

 TIP: 26 of the logged offenses dont fail the build.
      Run herb-lint --log-level=error to stop logging them, or set linter.logLevel in your .herb.yml.
      To start enforcing them instead, set linter.failLevel to hint in your .herb.yml.
```

Para el punto de partido corregí todo pero en alguna oportunidad futura evaluaré si quiero que los warnings rompan el build.

### En el CI

```yaml
herb:
	name: Herb
	runs-on: ubuntu-24.04
	steps:
		- uses: actions/checkout@v7
		- uses: actions/setup-node@v7
			with:
				node-version-file: .nvmrc
		- name: Lint HTML+ERB templates
			run: npx --yes @herb-tools/linter@0.11.2
```

La documentación indica que con un archivo `package.json` se puede instalar pero también ofrece la forma one-off con `npx` para no tener que instalar nada. Si en un futuro se vuelven lentos los builds, podría pasar a una versión instalada.
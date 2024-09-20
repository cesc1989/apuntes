# Body for Pull Request and final Commit

Cosas que debo destacar en el cuerpo del PR de AppBlend Milestone 2 y/o en el cuerpo del commit más bien.

- desactivar logs de active-record al correr pruebas
- Hay que usar los time formats explícitamente
- `Rails.application.config.active_support.use_rfc4122_namespaced_uuids` en falso en los defaults de 7.0
- assets precompile con yarn install

# Pull Request Body

Upgrade to Rails 7.0.4 for AppBlend.

> Version 7.0.4 because that's the version Clinical Dashboard and Patient Self Report are.

## Notable Changes

- Set `Rails.application.config.active_support.use_rfc4122_namespaced_uuids` to false.
	- Else it causes `Only UUIDs are valid namespace identifiers` error.
- Change log format in tests to `info` else it generates TONS of log lines when creating records.
- Any date / datetime interpolation needs to use `to_fs(FORMAT)`. The configuration in `config/initializers/date_time.rb` won't be catch automatically.
- Added rake task to run `assets:precompile` after `yarn:install`

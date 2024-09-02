# Body for Pull Request and final Commit

Cosas que debo destacar en el cuerpo del PR de AppBlend Milestone 2 y/o en el cuerpo del commit más bien.

- Cambios en `active_record_posgtres-constraints`
- Cambios en `batch-loader` (si se mantienen)
- Los callbacks en initializers
- desactivar logs de active-record al correr pruebas
- Hay que usar los time formats explícitamente
- audited ya no se usa el fork
- `Rails.application.config.active_support.use_rfc4122_namespaced_uuids` en falso en los defaults de 7.0
- assets precompile con yarn install

# Pull Request Body

Upgrade to Rails 7.0.4 for AppBlend.

> Version 7.0.4 because that's the version Clinical Dashboard and Patient Self Report are.

## Notable Changes

- Zeitwerk changes the way code in initializers is handled. I had to wrap lots of code in `Rails.application.config.after_initialize` callback.
- Set `Rails.application.config.active_support.use_rfc4122_namespaced_uuids` to false.
	- Else it causes `Only UUIDs are valid namespace identifiers` error.
- Change log format in tests to `info` else it generates TONS of log lines when creating records.
- Any date / datetime interpolation needs to use `to_fs(FORMAT)`. The configuration in `config/initializers/date_time.rb` won't be catch automatically.
- Added rake task to run `assets:precompile` after `yarn:install`
 
**Forked gems updated**

- Changes in `active_record_posgtres-constraints`. [See here](https://github.com/lunacare/active_record-postgres-constraints/commit/746999a68a1e0fbfa6a860cdfc262cd1c44ea41e).
- Changes in `batch-loader`. [See here](https://github.com/lunacare/batch-loader/commit/4a502c3f35fa7691c169640f0bc16a24426e2e67).
- Change audited to use source instead of fork

**Changes in tests**

Here I need confirmation whether the change is valid. I've tagged people who modified the files in question.

- spec/requests/graphql/performance/database_query_count_spec.rb
	- Tag: Anthony / Ryan
- app/models/therapist.rb && spec/models/therapist_spec.rb
	- Tag: Michael Shick
- app/services/scheduling_boundaries_calculator.rb
	- Tag: Anthony

## To dos

- [  ] Merge to Alpha
- [  ] QA in Alpha


# Commit Body
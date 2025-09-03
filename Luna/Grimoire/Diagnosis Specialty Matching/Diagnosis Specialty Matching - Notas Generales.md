# Notas Generales de Desarrollo

## ¿Qué diferencia hay entre `SchemaFactory` y `CaseCreationFactory`?

`SchemaFactory` crea el registro en la base de datos. En cambio `CaseCreationFactory` genera datos para el workflow (no necesariamente guardados en la BD).

Esto lo vi para cuando escribía las pruebas para POW y CCW. Había que definir una fecha de nacimiento para que el paciente fuera de edad menor a 65 años para que las pruebas fueran estables.

En `test/grimoire/case_creation/saga_builder_test.exs`:
```erlang
personal_info =
      CaseCreationFactory.build(:personal_info,
        region_id: region.id,
        date_of_birth: ~D[1990-01-01],
        address: CaseCreationFactory.build(:address, zip_code: zip_code.number)
      )
```

En `test/grimoire/patient_onboarding/saga_builder_test.exs`:
```erlang
personal_info =
      PatientOnboardingFactory.build(:personal_info,
        region_id: region.id,
        date_of_birth: ~D[1990-01-01],
        address: PatientOnboardingFactory.build(:address, zip_code: zip_code.number)
      )
```
# ¿Qué son Fax y Violation Rules

Etiquetas: #luna_help_desk 

Según Claudio:
- POC faxing logic: When POCs are faxed to physicians
- Violation logic: When compliance is notified of violations

Para los casos de Direct Access, estas reglas son usadas en las clases `PlansOfCare::DirectAccess::ViolationCalculator` para Violation Logic y en `PlansOfCare::DirectAccess::MessageCalculator`  para Fax Rules.

## Fax Rules

Este se rige por los valores de la llave `fax_rules`. A la fecha, toda configuración que tiene esta llave tendrá esta pareja:
```
anchor: initial_visit
```

Para la llave `trigger` los posibles valores para `strategy` son:

1. `chart_signature`: Fax immediately when chart is signed
2. `days_elapsed`: Fax after N days from anchor
3. `days_or_visits_elapsed`: Fax after N days OR M visits (whichever comes first)

> [!Tip]
> Cuando hay `strategy: chart_signature` no suele haber la key `days` por lo que se explica antes.

## Violation Rules

Se rige por los valores de la llave `violation_rules`. Cuando existe puede tener las llaves anidadas `pending` y `violation`.

> [!Tip]
> El equipo de Luna suele referirse a `pending` como "pending" y a `violation` como "yes".

# Composición General de PlansOfCare

El namespace `PlanOfCare` se compone de varios submodulos para dar cobertura a diferentes orígenes de POCs.

```
app/services/plans_of_care/
├── base
│   └── digital_signature_pdf_writer.rb
├── direct_access
│   ├── digital_signature_pdf_writer.rb
│   ├── generator.rb
│   ├── message_calculator.rb
│   └── violation_calculator.rb
├── fax_on_demand
│   ├── generator.rb
│   ├── message_calculator.rb
│   └── violation_calculator.rb
├── medicare
│   ├── digital_signature_pdf_writer.rb
│   ├── generator.rb
│   ├── message_calculator.rb
│   └── violation_calculator.rb
├── payer
│   ├── digital_signature_pdf_writer.rb
│   ├── generator.rb
│   ├── message_calculator.rb
│   └── violation_calculator.rb
├── plan_of_care_fax_pdf_service.rb
├── referred
│   ├── generator.rb
│   ├── message_calculator.rb
│   └── violation_calculator.rb
└── workers_comp
    ├── digital_signature_pdf_writer.rb
    ├── generator.rb
    ├── message_calculator.rb
    └── violation_calculator.rb
```

Donde cada clase del submodulo define diferentes formas de calcular las reglas de Fax y Violation.
# Consideraciones de Mezclar las apps como Engines, usando Packwerk o solo Namespaces

Ideas y comentarios sobre cómo abordar la migración de Patient Self Report, Clinical Dashboard y Therapist Signup a Edge.

Enlaces de interés:

- [Rails Engines](https://guides.rubyonrails.org/engines.html)
- [Packwerk](https://github.com/Shopify/packwerk)

# Engines o Packwerk

Leyendo un poco las guías de los Rails Engines y sobre Packwerk, creo que son un poco más sofisticados de lo que estoy buscando para mezclar los proyectos. Además, con la limitante de tiempo, usar alguna de estas alternativas podría convertirse en un dolor de cabeza y ser una mala decisión.

Además, los engines son algo poco usado en Edge, lo mismo que Packwerk. Entonces se pierde la ventaja de todos los devs que están trabajando Rails vanilla, por así decirle.

# Solo Namespaces

Mi alternativa inicial contempla solo usar namespaces para separar modelos, controladores, servicios y demás carpetas que llegarán de los proyectos a migrar. Para Patient Self Report definiría un namespace `PatientSelfReport` que correspondería con una carpeta `patient_self_report` en los subdirectorios donde convenga.

Luciría así, por ejemplo:
```
app/
├── models/
│   ├── shadow_user.rb
│   └── patient_self_report/
│       ├── aggravating_activity.rb
│       └── pain_spot.rb
├── controllers/
├── exceptions/
├── forms/
├── services/
│   ├── billing/
│   │   └── evaluator.rb
│   └── patient_self_report/
│       └── formatters/
│           └── set_date.rb
├── workers/
└── lib/
    ├── faraday/
    │   └── request.rb
    └── patient_self_report/
        ├── previous_progress_form.rb
        └── section_builder/
            └── builder.rb
```

Con esto podré separar la aplicación, tal y cómo llega desde su repo original en una subcarpeta definida y en un modulo bien definido para contener todo y a su vez facilitar encontrar todo lo relacionado al proyecto una vez esté dentro de Edge.

## La forma a seguir ⭐️

Creo que seguir esta forma de namespace con subcarpetas es más sencillo porque es más fácil de implementar y más fácil de continuar para otros devs. Además, esta misma estructura la puedo llevar a la carpeta de pruebas en RSpec.

Siguiendo esta forma, cuando esté ejecutando este milestone, solo tendría que:

- crear la sub carpeta `patient_self_report`
- copiar los archivos
	- la idea aquí es hacer un PR por cada carpeta dentro de `app` del proyecto a migrar
- actualizar las clases para que estén prefijadas y anidadas por el nombre del módulo
- ajustar las pruebas del mismo modo anterior
- ejecutar las pruebas

**Importante**:
El único caso donde tengo que hacer algo diferente es para los controladores de los servicios que actualmente se comunican entre backends. ==Para esos tengo que hacer el paso extra de cambiar el llamado al endpoint por un llamado a la clase==.
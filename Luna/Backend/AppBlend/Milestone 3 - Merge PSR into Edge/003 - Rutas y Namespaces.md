# Rutas y Namespaces: Cambios de URLs base

> [!Important]
> Las URL base de los servicios que apuntan a Forms Backend van a cambiar.
> 
> Ejemplo: en Forms Frontend, la URL base tendría que agregarle el sufijo `patient_self_report`. Así:
> `https://patient-self-report.alpha.getluna.com/patient_self_report`

# Proyectos que conecta con Forms Backend y las URLs base

Que yo sepa, los proyectos que se conectan a Forms Backend son:

1. Forms Frontend
2. Marketplace
3. Edge
4. Mobile Progress Forms app


# Rutas con la opción `path`  para determinar la URL base

Al importar las rutas desde el repo `patient-forms-backend`, las URLs quedan prefijadas por el namespace `patient_self_report`. Ejemplo:
```
Prefix: patient_self_report_api_diseases
Verb: GET
URI Pattern: /patient_self_report/diseases(.:format)
Controller#Action: patient_self_report/api/v1/diseases#index {:format=>"json"}
```

En cambio, en `patient-forms-backend` se muestra así:
```
Prefix: api_diseases
Verb: GET
URI Pattern: /diseases(.:format)
Controller#Action: api/v1/diseases#index {:format=>"json"}
```

Si lo dejo tal y cómo está al ser importado habrá que hacer cambio en los servicios que conectan con el Forms Backend porque la URL base cambiaría para tener el prefijo `patient_self_report`.

> [!Note]
> Guías de Rails sobre las rutas -> https://guides.rubyonrails.org/v7.0/routing.html#controller-namespaces-and-routing
> 
> Atención a la sección que muestra el uso de `path`

Entonces aquí lo mejor es quitarle ese prefijo al namespace usando `scope` o la opción `path`.

> [!Warning]
> Creo que no es necesario hacer nada más en particular. Lo creo que por si miramos las rutas `patients/:uuid` y `patient_forms` (bloque de código abajo) no se evidencia en que forma podrían colisionar con cualquier posible ruta de Edge para algún controlador `patients`.

Rutas `patient_forms` y `patients/:uuid`
```ruby
/patient_forms

/patients/:patient_internal_id/forms/:uuid(.:format)
/patients/:patient_internal_id/forms/:uuid(.:format)
/patients/:patient_internal_id/forms/:uuid(.:format)
/patients/:patient_internal_id/forms/:uuid(.:format)
/patients/:patient_internal_id/form_status/:status_uuid(.:format)
/patients/:patient_internal_id/form_results/:results_uuid(.:format)
/patients/:patient_internal_id/intake_forms(.:format)
/patients/:patient_internal_id/intake_forms/:id(.:format)
/patients/:patient_internal_id/request_intake_forms(.:format)
/patients/:patient_internal_id/summaries/:summary_uuid(.:format)
/patients/:patient_internal_id/reset/:uuid(.:format)
/patients/:patient_internal_id/reset/:uuid(.:format)
/patients/:internal_id(.:format)
/patients/:internal_id(.:format)
/patients/:internal_id(.:format)
```
# AWS Single Sign-On - Apuntes

Con la migración al nuevo entorno AWS, el ingreso se cambió de cuenta normal a [SSO](https://aws.amazon.com/es/what-is/sso/). Esto trajo beneficios a nivel de control de acceso para los DevOps pero para nosotros complica un poco las cosas.

> Docs en Confluence → https://lunacare.atlassian.net/wiki/spaces/ENG/pages/1499758714/Dev+Infra+-+AWS+Console+CLI+Quickstart

Tuve que cambiar varias cosas:

- Actualizar aws-cli a la versión 2 [+Install / Uninstall aws cli in MacOS](https://paper.dropbox.com/doc/Install-Uninstall-aws-cli-in-MacOS-NlX8Q6D9cVtVkAfyiHiD8) 
- Dejar de usar el comando `setaws`
    - Sigue habiendo perfiles pero las credenciales vencen más rápido así que no tiene sentido
- Ahora inicio sesión en AWS en esta url https://getluna.awsapps.com/start#/
- Para acceder mediante CLI necesito correr el comando `aws sso login --profile alpha`

```bash
$ aws sso login --profile alpha
Attempting to automatically open the SSO authorization page in your default browser.
If the browser does not open or you wish to use a different device to authorize this request, open the following URL:

https://device.sso.us-east-2.amazonaws.com/

Then enter the code:

VXVS-CKRT
Successfully logged into Start URL: https://getluna.awsapps.com/start#/
```

- Las llaves para acceso por código están en esa página
    - Se vencen cada 1 hora [según la documentación](https://docs.aws.amazon.com/singlesignon/latest/userguide/howtogetcredentials.html?icmpid=docs_sso_user_portal). Modificable a 12 horas.

> By default, credentials retrieved through the **AWS access portal are valid for 1 hour**. You can change this value up to 12 hours. Once you have completed this procedure, you can run any AWS CLI command (that your administrator has given you access to) **until those temporary credentials have expired**.

# ¿Cómo saber si las llaves de acceso caducaron?

Cuando se ejecute algún código que necesite el acceso a AWS saldrá este mensaje de error:
```
The security token included in the request is expired
```

Ejemplo:

Al hacer una petición:
```bash
Started POST "/v1/age_distribution" for ::1 at 2023-01-11 09:54:13 -0500
Processing by Api::V1::AgeDistributionController#create as JSON
	Parameters: {"data"=>{"doctors"=>["7e6fe728-a2f8-4e75-9935-cf2384999385"], "partners"=>[], "clinics"=>[]}, "age_distribution"=>{"data"=>{"doctors"=>["7e6fe728-a2f8-4e75-9935-cf2384999385"], "partners"=>[], "clinics"=>[]}}}
	ExecutionId Load (0.4ms)  SELECT "execution_ids".* FROM "execution_ids" WHERE "execution_ids"."chart_name" = $1 ORDER BY "execution_ids"."created_at" DESC LIMIT $2  [["chart_name", "age_distribution"], ["LIMIT", 1]]
	↳ app/services/age_distribution_chart_service.rb:9:in `initialize'
The security token included in the request is invalid.
{:service_class=>"AgeDistributionChartService"}
Completed 200 OK in 811ms (Views: 0.3ms | ActiveRecord: 0.4ms | Allocations: 4923)
```

O en una rake task:
```bash
$ tail log/development.log 
The security token included in the request is expired
{}
undefined method `each' for false:FalseClass
{:finder_class=>"AthenaNamedQueryFinder::AgeDistributionAllTime", :query_name=>"prod_normalized_ad_all_time"}
missing required parameter params[:query_string]
{:requester_class=>"Requesters::AgeDistributionRequesterService", :query_string_found=>false, :query_name=>nil, :query_name_id=>nil}
Age Distribution Execution IDs: [nil, nil]
Finished request Age Distribution query execution in Athena
```


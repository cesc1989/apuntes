# Apuntes Apex Ciclo 012 - Seguridad en Super Menú

## Proteger Super Menú de bots y escaneres 🛡️

Estaba haciendo un cambio sencillo y al revisar el log de supermenu veo esto:
```
Started POST "/" for 34.85.70.203
Started POST "/" for 34.85.70.203
Started POST "/" for 34.85.70.203
Started GET "/.git/config" for 34.85.70.203
Started POST "/" for 34.85.70.203
Started GET "/.env" for 34.85.70.203
Started GET "/.env.local" for 34.85.70.203
Started GET "/.env.production" for 34.85.70.203
Started GET "/.env.staging" for 34.85.70.203
Started GET "/.env.development" for 34.85.70.203
Started GET "/.env.test" for 34.85.70.203
Started GET "/.env.remote" for 34.85.70.203
Started GET "/.env.bak" for 34.85.70.203
Started GET "/.env.backup" for 34.85.70.203
Started GET "/.env.save" for 34.85.70.203
Started GET "/.env.old" for 34.85.70.203
Started GET "/.env.sample" for 34.85.70.203
Started GET "/.env.example" for 34.85.70.203
Started GET "/.env.dev" for 34.85.70.203
Started GET "/.env.prod" for 34.85.70.203
Started GET "/.env.stage" for 34.85.70.203
Started GET "/.env.ci" for 34.85.70.203
Started GET "/.env.docker" for 34.85.70.203
Started GET "/.env.live" for 34.85.70.203
Started GET "/.env.preprod" for 34.85.70.203
Started GET "/.env.uat" for 34.85.70.203
Started GET "/.env.dist" for 34.85.70.203
Started GET "/.env.swp" for 34.85.70.203
Started GET "/.env~" for 34.85.70.203
Started GET "/.env1" for 34.85.70.203
Started GET "/.env2" for 34.85.70.203
Started GET "/.env_copy" for 34.85.70.203
Started GET "/.env.txt" for 34.85.70.203
Started GET "/.env.json" for 34.85.70.203
Started GET "/.env.yaml" for 34.85.70.203
Started GET "/.env.yml" for 34.85.70.203
Started GET "/app/.env" for 34.85.70.203
Started GET "/apps/.env" for 34.85.70.203
Started GET "/api/.env" for 34.85.70.203
Started GET "/web/.env" for 34.85.70.203
Started GET "/site/.env" for 34.85.70.203
Started GET "/public/.env" for 34.85.70.203
Started GET "/admin/.env" for 34.85.70.203
Started GET "/backend/.env" for 34.85.70.203
Started GET "/server/.env" for 34.85.70.203
Started GET "/frontend/.env" for 34.85.70.203
Started GET "/src/.env" for 34.85.70.203
Started GET "/core/.env" for 34.85.70.203
Started GET "/core/app/.env" for 34.85.70.203
Started GET "/config/.env" for 34.85.70.203
Started GET "/private/.env" for 34.85.70.203
Started GET "/application/.env" for 34.85.70.203
Started GET "/bootstrap/.env" for 34.85.70.203
Started GET "/database/.env" for 34.85.70.203
Started GET "/storage/.env" for 34.85.70.203
Started GET "/var/www/.env" for 34.85.70.203
Started GET "/var/www/html/.env" for 34.85.70.203
Started GET "/current/.env" for 34.85.70.203
Started GET "/release/.env" for 34.85.70.203
Started GET "/releases/.env" for 34.85.70.203
Started GET "/shared/.env" for 34.85.70.203
Started GET "/deploy/.env" for 34.85.70.203
Started GET "/build/.env" for 34.85.70.203
Started GET "/dist/.env" for 34.85.70.203
Started GET "/public_html/.env" for 34.85.70.203
Started GET "/htdocs/.env" for 34.85.70.203
Started GET "/www/.env" for 34.85.70.203
Started GET "/html/.env" for 34.85.70.203
Started GET "/live/.env" for 34.85.70.203
Started GET "/prod/.env" for 34.85.70.203
Started GET "/dev/.env" for 34.85.70.203
Started GET "/staging/.env" for 34.85.70.203
Started GET "/opt/.env" for 34.85.70.203
Started GET "/laravel/.env" for 34.85.70.203
Started GET "/symfony/.env" for 34.85.70.203
Started GET "/wordpress/.env" for 34.85.70.203
Started GET "/wp/.env" for 34.85.70.203
Started GET "/cms/.env" for 34.85.70.203
Started GET "/drupal/.env" for 34.85.70.203
Started GET "/joomla/.env" for 34.85.70.203
Started GET "/magento/.env" for 34.85.70.203
Started GET "/shopify/.env" for 34.85.70.203
Started GET "/prestashop/.env" for 34.85.70.203
Started GET "/codeigniter/.env" for 34.85.70.203
Started GET "/cakephp/.env" for 34.85.70.203
Started GET "/zend/.env" for 34.85.70.203
Started GET "/yii/.env" for 34.85.70.203
Started GET "/laravel5/.env" for 34.85.70.203
Started GET "/v1/.env" for 34.85.70.203
Started GET "/v2/.env" for 34.85.70.203
Started GET "/v3/.env" for 34.85.70.203
Started GET "/api/v1/.env" for 34.85.70.203
Started GET "/api/v2/.env" for 34.85.70.203
Started GET "/rest/.env" for 34.85.70.203
Started GET "/graphql/.env" for 34.85.70.203
Started GET "/gateway/.env" for 34.85.70.203
Started GET "/microservice/.env" for 34.85.70.203
Started GET "/service/.env" for 34.85.70.203
Started GET "/api/v3/.env" for 34.85.70.203
Started GET "/api/dev/.env" for 34.85.70.203
Started GET "/api/staging/.env" for 34.85.70.203
Started GET "/vendor/.env" for 34.85.70.203
Started GET "/lib/.env" for 34.85.70.203
Started GET "/resources/.env" for 34.85.70.203
Started GET "/uploads/.env" for 34.85.70.203
Started GET "/internal/.env" for 34.85.70.203
Started GET "/tools/.env" for 34.85.70.203
Started GET "/scripts/.env" for 34.85.70.203
Started GET "/bin/.env" for 34.85.70.203
Started GET "/sbin/.env" for 34.85.70.203
Started GET "/local/.env" for 34.85.70.203
Started GET "/portal/.env" for 34.85.70.203
Started GET "/dashboard/.env" for 34.85.70.203
Started GET "/panel/.env" for 34.85.70.203
Started GET "/crm/.env" for 34.85.70.203
Started GET "/erp/.env" for 34.85.70.203
Started GET "/shop/.env" for 34.85.70.203
Started GET "/store/.env" for 34.85.70.203
Started GET "/saas/.env" for 34.85.70.203
Started GET "/client/.env" for 34.85.70.203
Started GET "/project/.env" for 34.85.70.203
Started GET "/admin-panel/.env" for 34.85.70.203
Started GET "/control-panel/.env" for 34.85.70.203
Started GET "/user-panel/.env" for 34.85.70.203
Started GET "/node/.env" for 34.85.70.203
Started GET "/express/.env" for 34.85.70.203
Started GET "/next/.env" for 34.85.70.203
Started GET "/nuxt/.env" for 34.85.70.203
Started GET "/nest/.env" for 34.85.70.203
Started GET "/react/.env" for 34.85.70.203
Started GET "/vue/.env" for 34.85.70.203
Started GET "/angular/.env" for 34.85.70.203
Started GET "/svelte/.env" for 34.85.70.203
Started GET "/vite/.env" for 34.85.70.203
Started GET "/backup/.env" for 34.85.70.203
Started GET "/backups/.env" for 34.85.70.203
Started GET "/old/.env" for 34.85.70.203
Started GET "/tmp/.env" for 34.85.70.203
Started GET "/temp/.env" for 34.85.70.203
Started GET "/lab/.env" for 34.85.70.203
Started GET "/cronlab/.env" for 34.85.70.203
Started GET "/cron/.env" for 34.85.70.203
Started GET "/en/.env" for 34.85.70.203
Started GET "/administrator/.env" for 34.85.70.203
Started GET "/psnlink/.env" for 34.85.70.203
Started GET "/exapi/.env" for 34.85.70.203
Started GET "/sitemaps/.env" for 34.85.70.203
Started GET "/.env.backup1" for 34.85.70.203
Started GET "/.env.backup2" for 34.85.70.203
Started GET "/logs/.env" for 34.85.70.203
Started GET "/cache/.env" for 34.85.70.203
Started GET "/mailer/.env" for 34.85.70.203
Started GET "/mail/.env" for 34.85.70.203
Started GET "/email/.env" for 34.85.70.203
Started GET "/smtp/.env" for 34.85.70.203
Started GET "/mailing/.env" for 34.85.70.203
Started GET "/notifications/.env" for 34.85.70.203
Started GET "/notify/.env" for 34.85.70.203
Started GET "/sender/.env" for 34.85.70.203
Started GET "/campaign/.env" for 34.85.70.203
Started GET "/newsletter/.env" for 34.85.70.203
Started GET "/ses/.env" for 34.85.70.203
Started GET "/sendgrid/.env" for 34.85.70.203
Started GET "/sparkpost/.env" for 34.85.70.203
Started GET "/postmark/.env" for 34.85.70.203
Started GET "/mailgun/.env" for 34.85.70.203
Started GET "/mandrill/.env" for 34.85.70.203
Started GET "/mailjet/.env" for 34.85.70.203
Started GET "/brevo/.env" for 34.85.70.203
Started GET "/transactional/.env" for 34.85.70.203
Started GET "/bulk/.env" for 34.85.70.203
Started GET "/aws/.env" for 34.85.70.203
Started GET "/azure/.env" for 34.85.70.203
Started GET "/gcp/.env" for 34.85.70.203
Started GET "/cloud/.env" for 34.85.70.203
Started GET "/infrastructure/.env" for 34.85.70.203
Started GET "/docker/.env" for 34.85.70.203
Started GET "/k8s/.env" for 34.85.70.203
Started GET "/kubernetes/.env" for 34.85.70.203
Started GET "/terraform/.env" for 34.85.70.203
Started GET "/ansible/.env" for 34.85.70.203
Started GET "/.git/.env" for 34.85.70.203
Started GET "/ci/.env" for 34.85.70.203
Started GET "/cd/.env" for 34.85.70.203
Started GET "/jenkins/.env" for 34.85.70.203
Started GET "/gitlab/.env" for 34.85.70.203
Started GET "/github/.env" for 34.85.70.203
Started GET "/actions/.env" for 34.85.70.203
Started GET "/circleci/.env" for 34.85.70.203
Started GET "/travis/.env" for 34.85.70.203
Started GET "/buildkite/.env" for 34.85.70.203
Started GET "/mysql/.env" for 34.85.70.203
Started GET "/postgres/.env" for 34.85.70.203
Started GET "/mongodb/.env" for 34.85.70.203
Started GET "/redis/.env" for 34.85.70.203
Started GET "/elasticsearch/.env" for 34.85.70.203
Started GET "/rabbitmq/.env" for 34.85.70.203
Started GET "/kafka/.env" for 34.85.70.203
Started GET "/queue/.env" for 34.85.70.203
Started GET "/worker/.env" for 34.85.70.203
Started GET "/job/.env" for 34.85.70.203
Started GET "/test/.env" for 34.85.70.203
Started GET "/qa/.env" for 34.85.70.203
Started GET "/preview/.env" for 34.85.70.203
Started GET "/beta/.env" for 34.85.70.203
Started GET "/uat/.env" for 34.85.70.203
Started GET "/stage/.env" for 34.85.70.203
Started GET "/development/.env" for 34.85.70.203
Started GET "/production/.env" for 34.85.70.203
Started GET "/config/app/.env" for 34.85.70.203
Started GET "/phpinfo.php" for 34.85.70.203
Started GET "/info.php" for 34.85.70.203
Started GET "/php.php" for 34.85.70.203
Started GET "/i.php" for 34.85.70.203
Started GET "/pi.php" for 34.85.70.203
Started GET "/pinfo.php" for 34.85.70.203
Started GET "/test.php" for 34.85.70.203
Started GET "/phpinfo" for 34.85.70.203
Started GET "/p.php" for 34.85.70.203
Started GET "/debug.php" for 34.85.70.203
Started GET "/admin/phpinfo.php" for 34.85.70.203
Started GET "/test/phpinfo.php" for 34.85.70.203
Started GET "/dev/phpinfo.php" for 34.85.70.203
Started GET "/old/phpinfo.php" for 34.85.70.203
Started GET "/tmp/phpinfo.php" for 34.85.70.203
Started GET "/public/phpinfo.php" for 34.85.70.203
Started GET "/info" for 34.85.70.203
Started GET "/php-info.php" for 34.85.70.203
Started GET "/phpversion.php" for 34.85.70.203
Started GET "/_phpinfo.php" for 34.85.70.203
Started GET "/old_phpinfo.php" for 34.85.70.203
Started GET "/server-info.php" for 34.85.70.203
Started GET "/server-status.php" for 34.85.70.203
Started GET "/_profiler/phpinfo" for 34.85.70.203
Started GET "/_environment" for 34.85.70.203
Started GET "/webroot/index.php/_environment" for 34.85.70.203
Started GET "/mail/phpinfo.php" for 34.85.70.203
Started GET "/cpanel/phpinfo.php" for 34.85.70.203
Started GET "/hosting/phpinfo.php" for 34.85.70.203
Started GET "/webmail/phpinfo.php" for 34.85.70.203
Started GET "/smtp/phpinfo.php" for 34.85.70.203
Started GET "/phpinfo.php.bak" for 34.85.70.203
Started GET "/phpinfo.php.old" for 34.85.70.203
Started GET "/phpinfo.php~" for 34.85.70.203
Started GET "/info.php.bak" for 34.85.70.203
Started GET "/phpinfo.php.save" for 34.85.70.203
Started GET "/staging/phpinfo.php" for 34.85.70.203
Started GET "/beta/phpinfo.php" for 34.85.70.203
Started GET "/uat/phpinfo.php" for 34.85.70.203
Started GET "/qa/phpinfo.php" for 34.85.70.203
Started GET "/preview/phpinfo.php" for 34.85.70.203
Started GET "/www/phpinfo.php" for 34.85.70.203
Started GET "/htdocs/phpinfo.php" for 34.85.70.203
Started GET "/public_html/phpinfo.php" for 34.85.70.203
Started GET "/site/phpinfo.php" for 34.85.70.203
Started GET "/docs/phpinfo.php" for 34.85.70.203
Started GET "/wp-admin/phpinfo.php" for 34.85.70.203
Started GET "/administrator/phpinfo.php" for 34.85.70.203
Started GET "/core/phpinfo.php" for 34.85.70.203
Started GET "/includes/phpinfo.php" for 34.85.70.203
Started GET "/service-account.json" for 34.85.70.203
Started GET "/sa.json" for 34.85.70.203
Started GET "/gcp-key.json" for 34.85.70.203
Started GET "/gcp-credentials.json" for 34.85.70.203
Started GET "/gcp-sa.json" for 34.85.70.203
Started GET "/credentials.json" for 34.85.70.203
Started GET "/google-credentials.json" for 34.85.70.203
Started GET "/google-key.json" for 34.85.70.203
Started GET "/.config/gcloud/application_default_credentials.json" for 34.85.70.203
Started GET "/application_default_credentials.json" for 34.85.70.203
Started GET "/key.json" for 34.85.70.203
Started GET "/keyfile.json" for 34.85.70.203
Started GET "/firebase-adminsdk.json" for 34.85.70.203
Started GET "/firebase-key.json" for 34.85.70.203

```

> [!Note]
> Los saqué del log con `awk`:
> ```
> grep Started ~/Downloads/supermenulogprod.log | awk '{print $8, $9, $10, $11, $12}'
> ```

Así que le pedí a Claudio revisar y armar un plan para cubrir eso.

### rack-attack ⚔

Se hicieron los cambios en este PR: https://github.com/devaspros/supermenu/pull/24

#### Pruebas para confirmar que protege

##### Prueba #1

Petición
```
curl https://supermenu.devaspros.com/newsletter/.env
Forbidden
```

Log:
```
W, [2026-10-30T18:47:55.625263 #129414]  WARN -- : [0e25b599-0398-4d37-be7b-f8594ed89a6b] [Rack::Attack] blocklist scanners 191.110.52.188 GET /newsletter/.env
```

##### Prueba #2

Petición
```
curl https://supermenu.devaspros.com/.env.local
Forbidden
```

Log:
```
W, [2026-10-30T18:47:55.625263 #129414]  WARN -- : [0e25b599-0398-4d37-be7b-f8594ed89a6b] [Rack::Attack] blocklist scanners 191.110.52.188 GET /.env.local
```



### nginx


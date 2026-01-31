# DevOps & Heroku

## DevOps

- No `Access-Control-Allow-Origin` header is present on the requested resource: [1](https://github.com/cyu/rack-cors/issues/117) y [2](https://gist.github.com/dhoelzgen/cd7126b8652229d32eb4)
- CORS preflight request [MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/OPTIONS) - [More about CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS)
- How to create a Vagrant Box for Rails & PostgreSQL development. Provisioned with bash scripts
- How to create a Vagrant Box for Rails & PostgreSQL development. Provisioned with Ansible
- Sidekiq as an Upstart service [Stack Overflow](https://stackoverflow.com/questions/25342465/booting-up-sidekiq-with-upstart) - [Mike Perham](http://www.mikeperham.com/2015/07/16/sidekiq-and-upstart/)


## Heroku

- Cómo hacer despliegues en Heroku desde BitBucket: [Atlassian](https://confluence.atlassian.com/bitbucket/deploy-to-heroku-872013667.html)
- Especificar versión de Ruby en el Gemfile puede ayudar con errores de despliegue: [Heroku Devecenter](https://devcenter.heroku.com/articles/ruby-versions)
- Recuerda configurar Puma para proyectos en Heroku junto con un Procfile: [Heroku Dev Center](https://devcenter.heroku.com/articles/deploying-rails-applications-with-the-puma-web-server#adding-puma-to-your-application)
- A tener en cuenta sobre ambientes diferentes a producción según Heroku: [Dev Center](https://devcenter.heroku.com/articles/deploying-to-a-custom-rails-environment)


## Cómo configurar varios perfiles en el AWS CLI

[Artículo](https://medium.com/@Rajendrasinh/multiple-user-profiles-in-aws-cli-64d5df3d181d).

Así debe verse `~/.aws/credentials`
```
[default]
aws_access_key_id=KEY
aws_secret_access_key=SECRET

[user1]
aws_access_key_id=KEY
aws_secret_access_key=SECRET
```


Así debe verse `~/.aws/config`
```bash
[default]
region=us-west-2
output=json

[profile user1]
region=us-east-1
output=json
```

Y se ejecuta el comando `aws` indicando la bandera del perfil:
```bash
aws ec2 describe-instances --profile myprofile
```


# Apuntes Ciclo 14 - Cash Flow

# Despliegue solo en rama especificada en Github Actions

Se usa la sentencia `if` para el `job` que se quiere ejecutar la condición.

    name: my workflow
    on: push
    jobs:
      test:
        runs-on: ubuntu-latest
        steps:
          - name: Execute tests
            run: exit 0
      deploy:
        runs-on: ubuntu-latest
        needs: test
        if: github.ref == 'refs/heads/master'
        steps:
          - name: Deploy app
            run: exit 0
    

Encontrado en [Stack Overflow](https://stackoverflow.com/questions/58139406/only-run-job-on-specific-branch-with-github-actions/58142412#58142412).


# Cargar favicon con helper de Rails

Rails tiene el helper `favicon_link_tag` y sirve para cargar una imagen en la carpeta assets.

    <%= favicon_link_tag asset_path('image-name.png') %>

Encontrado [aquí](https://josephcardillo.medium.com/how-to-add-a-favicon-to-your-rails-app-9676336f7006).


# Usando Mega para backups

Así se instala en e servidor de Cashflow. Ubuntu 20.04 LTS. 64 bit.

    wget https://mega.nz/linux/repo/xUbuntu_20.04/amd64/megacmd-xUbuntu_20.04_amd64.deb && sudo apt install "$PWD/megacmd-xUbuntu_20.04_amd64.deb"


## Comandos de mega-cmd
- mega-help
- `mega-login email pass`
    - el password va en texto plano
- mega-ls
    - para listar archivos
- `mega-put cashflowdb-202312261703563201.tgz cashflow/db-backups`
    - para hacer el backup

Página de Mega CMD https://mega.io/es/cmd

Guía de Usuario https://github.com/meganz/MEGAcmd/blob/master/UserGuide.md


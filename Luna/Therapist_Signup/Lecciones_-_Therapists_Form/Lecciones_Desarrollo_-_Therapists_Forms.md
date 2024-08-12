# Lecciones Desarrollo - Therapists Forms

## Configuración por defecto del comando `git push`

Se puede configurar para que el comando envíe la rama sin necesidad de especificar que la cree en el upstream(repositorio)


    git push -u origin [BRANCH-NAME]

Con la configuración `git.config.push` [como se explica en la documentación](https://git-scm.com/docs/git-config/2.20.0#git-config-pushdefault):


    # ~/.gitconfig
    [push]
      default = current


## Usando el comando awk para extraer una porción de texto

En la cagada que hubo luego de correr el comando `rake therapists:sync_therapists_info_to_hubspot` donde se modificaron muchos valores que no debíamos, tocó identificar los correos afectados.

Ryan me compartió un archivo de texto que era todo el log de rails. De eso, pude extraer las líneas que necesitaba.

Pasamos de esto:

      Therapist Load (772.5ms)  SELECT "therapists".* FROM "therapists" WHERE "therapists"."hubspot_id" IS NOT NULL ORDER BY "therapists"."id" ASC LIMIT $1  [["LIMIT", 1000]]
    I, [2022-05-18T19:30:12.389874 #2046]  INFO -- : Syncing Therapist ericarao@hotmail.com to Hubspot Contact: 105139901
    I, [2022-05-18T19:30:12.390010 #2046]  INFO -- : Running HubspotFormFilesService
    I, [2022-05-18T19:30:12.711087 #2046]  INFO -- : Running HubspotFormUrlService
    I, [2022-05-18T19:30:12.991079 #2046]  INFO -- : Running HubspotPersonalInformationService
    I, [2022-05-18T19:30:13.455034 #2046]  INFO -- : Running HubspotCredentialingService
    # (...)

Y luego con este comando:

    $ cat ~/Downloads/cred_logs.txt | grep "Syncing Therapist" >> ~/Downloads/affected.emails.txt

Pasamos a esto:

    I, [2022-05-18T19:30:12.389874 #2046]  INFO -- : Syncing Therapist ericarao@hotmail.com to Hubspot Contact: 105139901
    I, [2022-05-18T19:30:15.933514 #2046]  INFO -- : Syncing Therapist prdoucot@gmail.com to Hubspot Contact: 73631584
    I, [2022-05-18T19:30:19.577926 #2046]  INFO -- : Syncing Therapist lyannec13@gmail.com to Hubspot Contact: 75072051

Entonces, para poder extraer el email que era lo que interesaba, me valí de la [utilidad awk](https://www.howtogeek.com/562941/how-to-use-the-awk-command-on-linux/).


> By default, `awk` considers a field to be a string of characters surrounded by whitespace, the start of a line, or the end of a line. Fields are identified by a dollar sign (`$`) and a number. So, `$1` represents the first field, which we’ll use with the `print` action to print the first field.

Con este comando:

    $ cat ~/Downloads/affected.emails.txt | awk '{print $9}' >> ~/Downloads/emails.affected.txt

Pude lograr esto:

    ericarao@hotmail.com
    prdoucot@gmail.com
    lyannec13@gmail.com
    lauramizumoto7@gmail.com
    eileen_lansang@yahoo.com
    AdamPaicely@gmail.com
    tmalave4@hotmail.com

Y lograr un archivo con solo la parte del correo. ¡Fantástico!


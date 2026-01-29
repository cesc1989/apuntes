# Apuntes Ciclo 12 - Mantenimiento - Cash Flow

# Error de whenever al generar el cron

Lo está generando así:

    # Begin Whenever generated tasks for: /home/ubuntu/cashflow/app/config/schedule.rb at: 2023-08-15 02:58:36 +0000
    20 0 1 * * /bin/bash -l -c 'cd /home/ubuntu/cashflow/app && bundle exec bin/rails runner -e production '\''RecurrentExpenditures.create_monthly'\'''

Notese:

    '\''RecurrentExpenditures.create_monthly'\'''

Está escapando la instrucción. Es un error ya comentado de hace tiempo. Visto en estos issues:

- [Quotes gets replaced by escaped single quotes](https://github.com/javan/whenever/issues/709)
- [Issue when using runner](https://github.com/javan/whenever/issues/816)

La solución que comentan es usar una rake task en lugar de llamar a la clase directamente. Vamos a probar suerte con eso.


    rake "my:rake:task"




# Apuntes Base de Datos - Parte 02

# Actualizar una columna con diferentes valores para varios registros

Para actualizar el campo `exercises_web_token` de Accounts mediante SQL y que cada registro tenga su propio token diferente lo puedo hacer así:

    UPDATE accounts
    SET exercises_web_token = substr(md5(random()::text || clock_timestamp()::text), 1, 16)
    where id in (
      select acc.id
      from accounts acc
      inner join patients pat on pat.account_id = acc.id
      where acc.exercises_web_token is null
    );

Nótese que para poder efectuar sobre solo los Accounts que son de Patient tuve que hacer el subquery en la clausula IN para poder hacer el INNER JOIN.

Una alternativa a:

    substr(md5(random()::text || clock_timestamp()::text), 1, 16)

Es generar un UUID, así:

    uuid_generate_v4();
    # f47ac10b-58cc-4372-a567-0e02b2c3d479

Me lo dijo chatgpt [https://chat.openai.com/share/461bc1de-a0f0-4815-898f-7d92fe35e40c](https://chat.openai.com/share/461bc1de-a0f0-4815-898f-7d92fe35e40c)


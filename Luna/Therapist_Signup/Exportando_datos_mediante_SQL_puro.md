# Exportando datos mediante SQL puro
En Patient Forms y Dashboard he hecho los exports a Athena mediante ActiveRecord, sin embargo, Ryan sugiere mejor usar SQL puro. Ahora que se está exportando data de Credentialing a Tableau, vi la oportunidad perfecta para hacer este proceso.

Algunos apuntes a continuación.


# Usando Common Table Expressions de PostgreSQL

Conocidos como CTE, permiten hacer queries más complejas generando tablas que solo existen durante la ejecución de la consulta.

Este es el ejemplo con la query para exportar los datos de Credentialing:

```sql
 WITH therapist
AS (
	SELECT /* FIELDS */
	FROM therapists t
), immunization
AS (
	SELECT /* FIELDS */
	FROM immunizations i
	JOIN therapist t ON t.id = i.therapist_id
), credentialing
AS (
	SELECT /* FIELDS */
	FROM credentialing_informations c
	JOIN therapist t ON t.id = c.therapist_id
), preference
AS (
	SELECT /* FIELDS */
	FROM preferences p
	JOIN therapist t ON t.id = p.therapist_id
), npi_and_caqh
AS (
	SELECT /* FIELDS */
	FROM npi_and_caqh_applications n
	JOIN therapist t ON t.id = n.therapist_id
), payout
AS (
	SELECT /* FIELDS */
	FROM payouts pa
	JOIN therapist t ON t.id = pa.therapist_id
), therapist_summary
AS (
	SELECT t.id,
		/* FIELDS */
	FROM therapist t
	JOIN immunization i ON i.therapist_id = t.id
	JOIN credentialing c ON c.therapist_id = t.id
	JOIN preference p ON p.therapist_id = t.id
	JOIN npi_and_caqh n ON n.therapist_id = t.id
	JOIN payout pa ON pa.therapist_id = t.id
	WHERE t.form_completed = true
	AND t.form_completed_at IS NOT NULL
)
SELECT * FROM therapist_summary;
```

[Al respecto](https://www.postgresqltutorial.com/postgresql-tutorial/postgresql-cte/).

# ActiveRecord::Base.connection

En el proyecto Backend usan esto:

    ActiveRecord::Base.connection.raw_connection

Para obtener una conexión a la base de datos para ejecutar SQL puro.

Sin embargo, en Backend lo hacen es de esta forma:

    DatabaseSwitching.with_raw_read_connection do |connection|
    end

El módulo `DatabaseSwitching` lo que hace es definir una serie de métodos para leer la base de datos si hay réplica o solo primaria.

No quise traer toda esa complejidad a Credentialing y opté por hacer esto:

    ActiveRecord::Base.connection.raw_connection do |connection|
    end

Pero eso no funciona porque `ActiveRecord::Base.connection.raw_connection` no acepta bloque.

En el caso de Backend, el módulo `DatabaseSwitching` sí está configurado para recibir un bloque.

Al final me tocó hacerlo así:

    connection = ActiveRecord::Base.connection.raw_connection
    
    connection.send_query(sql_query)
    
    connection.set_single_row_mode

Sobre lo visto aquí:

- [raw_connection](https://api.rubyonrails.org/classes/ActiveRecord/ConnectionAdapters/AbstractAdapter.html#method-i-raw_connection)
- [connection](https://apidock.com/rails/ActiveRecord/ConnectionAdapters/ConnectionPool/connection)
- [Streaming results](https://blog.magrathealabs.com/fetching-millions-of-rows-from-postgresql-with-rails-70c0cec1b6f5)


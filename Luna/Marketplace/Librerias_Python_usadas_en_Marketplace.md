# Librerias Python usadas en Marketplace

# Flask

URL: https://flask.palletsprojects.com/en/3.0.x/
Descripción:

> Flask is a lightweight [WSGI](https://wsgi.readthedocs.io/) web application framework. It is designed to make getting started quick and easy, with the ability to scale up to complex applications. 
> 
> Flask offers suggestions, but doesn’t enforce any dependencies or project layout. It is up to the developer to choose the tools and libraries they want to use.

## Apuntes

# SQL Alchemy

URL: https://www.sqlalchemy.org/
Descripción:

> SQLAlchemy is the Python SQL toolkit and Object Relational Mapper that gives application developers the full power and flexibility of SQL.

## Apuntes

Hay partes donde se ven estas queries:
```python
session.query(models.TherapistSignUp).get(therapist_sign_up_id)
```

Están bien porque están escritas en la versión 1.0 como [mencionan aquí](https://docs.sqlalchemy.org/en/20/changelog/migration_20.html#migration-20-query-usage).

Siguiendo con la función query. Encuentro que:
```python
session.query(models.VisitPlan)
```

Es forma de trabajar en SQL Alchemy 1.0. Estos son los docs para [Query object](https://docs.sqlalchemy.org/en/20/orm/queryguide/query.html#the-query-object).

Ahí también encuentro el método `add_column()` el cual es usado en `windowed_query`:
```python
def windowed_query(query, column, window_size=1_000):
	"""Breaks a query into chunks on a given column."""
	single_entity = query.is_single_entity
	query = query.add_column(column).order_by(column)
```

La documentación de `query.add_column` dice:

> Add a column expression to the list of result columns to be returned.

Es para definir las columnas que queremos que sean devueltas en la consulta.

**Clausula IN**
[Docs](https://docs.sqlalchemy.org/en/20/orm/extensions/associationproxy.html#sqlalchemy.ext.associationproxy.AssociationProxyInstance.in_).

En versión 2.0
```python
stmt.where(column.in_([1, 2, 3]))
```

En versión 1.0
```python
session.query(MyUserClass).filter(MyUserClass.id.in_((123,456))).all()
```

As mentioned [here](https://stackoverflow.com/a/8603129/1407371).

**Hay que interactuar con los resultados de las consultas para ver los valores**
Según [esto](https://stackoverflow.com/a/46370887/1407371).

**relationship()**
[Docs](https://docs.sqlalchemy.org/en/20/orm/relationship_api.html#sqlalchemy.orm.relationship).

# Click

URL: https://click.palletsprojects.com/en/8.1.x/
Descripción:

> Click is a Python package for creating beautiful command line interfaces in a composable way with as little code as necessary. It’s the “Command Line Interface Creation Kit”. It’s highly configurable but comes with sensible defaults out of the box.

## Apuntes

Así se ejecutan los Click commands que están en la carpeta `app/marketplace/commands/`:
```bash
python -m marketplace.commands.google_drive list-folder-and-files-in-root-gdrive-folder
```

Con la forma:
```bash
python -m namespace.folder.file method-name
```

# Pendulum

URL: https://pendulum.eustace.io/
Descripción:

> Pendulum is a Python package to ease datetimes manipulation.
> It provides classes that are drop-in replacements for the native ones (they inherit from them).

## Crear períodos o rangos

[Docs](https://pendulum.eustace.io/docs/#period) en Pendulum.
Para lo de visit plan se usa mucho `Period`.

Al probar generar uno, esto obtengo:
```python
pendulum.Period(measure_from_day, min_possible_end_day)
<Period [2021-12-29 -> 2023-12-12]>
```

Hay muchas líneas haciendo iterando un período:
```python
for day in pendulum.Period(measure_from_day, min_possible_end_day):

######

effective_from_window = pendulum.Period(minimized_entry.effective_from, effective_from_until)
if day in effective_from_window:

######

for day in pendulum.Period(measure_from_day, max_possible_end_day):

######

effective_from_window = pendulum.Period(maximized_entry.effective_from, effective_from_until)
if day in effective_from_window:
```

Esto dice la documentación para iterar instancias de `Period`:

> You can also directly iterate over the `Period` instance, the unit will be `days` in this case:

```python
>>> for dt in period:
>>> 	print(dt)
```

# Attr

URL: https://www.attrs.org/
Descripción:

> attrs is the Python package that will bring back the **joy** of **writing classes** by relieving you from the drudgery of implementing object protocols

???

## Uso de attr.ib()

Docs → https://www.attrs.org/en/stable/api-attr.html#attr.ib
Descripción:

> Create a new attribute on a class.
> 
> Please consider using `[attrs.field](https://www.attrs.org/en/stable/api.html#attrs.field)` in new code (`attr.ib` will never go away, though).

Veo muchas partes donde se definen modelos o clases que usan esta función.
```python
@attr.s(frozen=True, slots=True)
class PhysicianAttached:
		"""Physician is attached to a care plan."""

		EVENT_TYPE = "physician_attached"

		physician_id = attr.ib()
		npi = attr.ib()
		first_name = attr.ib()
		last_name = attr.ib()
```

# Google API Python Client

URL: https://pypi.org/project/google-api-python-client/
Documentación inicial: https://github.com/googleapis/google-api-python-client/blob/main/docs/README.md

Estos son [los docs](https://googleapis.github.io/google-api-python-client/docs/epy/index.html) de la función `build` usada en `discover_client()`

```python
# app/marketplace/services/google_drive.py

def discover_client(minimum_storage_in_megabytes: int = 100) -> discovery.Resource:
		client_candidate = discovery.build("drive", "v3", cache_discovery=False, credentials=credentials)
```

Y luego se usa así:
```python
# app/marketplace/services/google_drive.py

def upload_file(file_name: str, file_path: str, parent_folder_ids: Iterable[str], target_mime_type: Optional[str] = None) -> GDriveFileInfo:
		file = discover_client().files().create(body=file_metadata, media_body=media, fields="*").execute()
```

¿De dónde sale la función `files()`?

No encuentro aún donde se define pero encontré [información sobre la API](https://developers.google.com/drive/api/reference/rest/v3?hl=es-419#rest-resource:-v3.files) de acceso a archivos en Google Drive.

Esta parece ser la explicación. ¿Metaprogramación?

> The service object is constructed with a function for every collection defined by the API. If the given API has a collection named `stamps`, you create the collection object like this:
    collection = service.stamps()


# Python en General

## pdb - The Python Debugger

URL: https://docs.python.org/3/library/pdb.html
Descripción:

> The module `[pdb](https://docs.python.org/3/library/pdb.html#module-pdb)` defines an interactive source code debugger for Python programs. It supports setting (conditional) breakpoints and single stepping at the source line level, inspection of stack frames, source code listing, and evaluation of arbitrary Python code in the context of any stack frame.

**Apuntes**
Dos formas de uso:
```python
import pdb; pdb.set_trace()
```

o así:
```python
breakpoint()
```

## str(value)

[Docs](https://docs.python.org/3/library/stdtypes.html#str).

Esta es la forma de convertir un objeto a su representación en string.

Por ejemplo, si tengo esto en Marketplace:
```python
appointment_ids = [plan.appointment_id for plan in visit_plans]
```

`appointment_ids` será esto:
```python
[UUID('dc5a85f9-92a7-47c0-9e71-2d10e8d086c2'), UUID('c4fc18f7-5947-4c89-a752-d126cae85309'), ...]
```

Por eso hay que hacerlo con `str()`:
```python
appointment_ids = [str(plan.appointment_id) for plan in visit_plans]

['dc5a85f9-92a7-47c0-9e71-2d10e8d086c2', 'c4fc18f7-5947-4c89-a752-d126cae85309', ...]
```

## list()

Tomado de [The Data Schools](https://thedataschools.com/python/funciones/list-funcion/).

> la función list() se utiliza para crear una nueva lista a partir de otro iterable, como una tupla, una cadena de texto, un rango o incluso otra lista. Esta función convierte el iterable en una lista, permitiéndote realizar operaciones específicas de lista en los elementos.

## filter()

[Docs](https://thedataschools.com/python/funciones/filter-funcion/).

> se utiliza para filtrar elementos de una secuencia (como una lista o una tupla) utilizando una función de filtro específica. Devuelve un iterable (un objeto iterable que puede ser convertido a una lista u otra estructura de datos) que contiene solo los elementos que cumplen con la condición definida en la función de filtro.

Sintaxis:
```python
filter(función, iterable)
```

En el caso de VisitPlan:
```python
list(filter(None, [current_entry]))
```

Aquí no se filtra nada. Para qué hacerlo de esa forma?

## sum()

Tomado de [The Data Schools](https://thedataschools.com/python/funciones/sum-funcion/).

> se utiliza para calcular la suma de los elementos de un iterable (como una lista, una tupla, un conjunto o un rango).

Sintaxis:
```python
sum(iterable, start)
```

Parámetros:

- iterable (obligatorio): El iterable del cual se desea calcular la suma.
- start (opcional): El valor inicial de la suma. Si se proporciona, se agregará al resultado final.

Este es un ejemplo que generó ChatGPT
```python
expected_visits_marginal_increase = sum(
		minimized_entry.frequency_per_day.lower
		for (start_date, end_date) in effective_date_ranges
		if start_date <= day <= end_date
)
```


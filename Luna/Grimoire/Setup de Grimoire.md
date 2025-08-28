# Configuración de Grimoire

Se configura con la herramienta asdf.

Comandos:
```bash
asdf plugin add erlang
asdf plugin add elixir
asdf plugin add nodejs
asdf plugin add direnv
```

Después:
```
asdf install
```

Le sigue:
```
mix deps.get
```

Continuamos con la configuración de las llaves de Oban:
```
mix hex.repo add oban https://getoban.pro/repo     --fetch-public-key PUBLIC_KEY     --auth-key AUTH_KEY
```

Sigue `mix setup`.

## mix setup

Para que `mix setup` sea exitoso debo correrlo de esta forma:
```bash
DEV_SEED_EMAIL="francisco.quintero@ideaware.co" mix ecto.setup
```

De no hacerlo voy a tener varios errores porque en el paso que pide ingresar el correo del usuario, a pesar de que hay prompt, el setup no permite interactuar:
```
Is your luna email: francisco.quintero@ideaware.co? [y/n] - but it never lets me input something

** (RuntimeError) could not lookup Ecto repo Grimoire.Repo because it was not started or it does not exist
      (ecto 3.12.5) lib/ecto/repo/registry.ex:22: Ecto.Repo.Registry.lookup/1
      (ecto 3.12.5) lib/ecto/repo/supervisor.ex:176: Ecto.Repo.Supervisor.tuplet/2
      (grimoire 0.1.0) lib/grimoire/repo.ex:3: Grimoire.Repo.get_by/3
      priv/repo/seeds/bootstrap_accounts_and_groups.exs:14: BootstrapAccountsAndGroups.seed/0
      priv/repo/seeds/bootstrap_accounts_and_groups.exs:138: (file)
      (elixir 1.18.3) lib/code.ex:1525: Code.require_file/2
      priv/repo/seeds.exs:13: (file)
```

## PostgreSQL

Necesita el rol `postgres` que exista y que pueda hacer `LOGIN`.
```
psql

francisco=# CREATE ROLE postgres SUPERUSER;
francisco=# ALTER ROLE postgres WITH LOGIN;
```

También necesitamos la extensión/plugin de postgis. Sino tira este error:
```bash
** (Postgrex.Error) ERROR 0A000 (feature_not_supported) extension "postgis" is not available

    hint: The extension must first be installed on the system where PostgreSQL is running.

Could not open extension control file "/opt/homebrew/opt/postgresql@16/share/postgresql@16/extension/postgis.control": No such file or directory.
    (ecto_sql 3.12.1) lib/ecto/adapters/sql.ex:1096: Ecto.Adapters.SQL.raise_sql_call_error/1
    (elixir 1.17.3) lib/enum.ex:1703: Enum."-map/2-lists^map/1-1-"/2
    (ecto_sql 3.12.1) lib/ecto/adapters/sql.ex:1203: Ecto.Adapters.SQL.execute_ddl/4
    (ecto_sql 3.12.1) lib/ecto/migration/runner.ex:348: Ecto.Migration.Runner.log_and_execute_ddl/3
    (elixir 1.17.3) lib/enum.ex:1703: Enum."-map/2-lists^map/1-1-"/2
    (ecto_sql 3.12.1) lib/ecto/migration/runner.ex:311: Ecto.Migration.Runner.perform_operation/3
    (stdlib 6.1) timer.erl:590: :timer.tc/2
    (ecto_sql 3.12.1) lib/ecto/migration/runner.ex:25: Ecto.Migration.Runner.run/8
```

### Postgis

En Macos Sillicon, hay que primero instalar estas dependencias de postgis:
```
HOMEBREW_NO_AUTO_UPDATE=1 brew install geos proj protobuf-c gdal
```

Luego:
```bash
curl -o /tmp/postgis-3.4.2.tar.gz https://download.osgeo.org/postgis/source/postgis-3.4.2.tar.gz

tar xzf /tmp/postgis-3.4.2.tar.gz -C /tmp

pushd /tmp/postgis-3.4.2

eval ./configure `pg_config --configure` --with-projdir=$(brew --cellar proj)/$(pkg-config --modversion proj) --with-pgconfig=$(which pg_config) --without-protobuf

make

make install

popd
```

Ver [[Configurando PostgreSQL para Grimoire]] para ver logs de cuándo los comandos son exitosos.

### Vector

Esto también se necesita...

```bash
cd /tmp
git clone --branch v0.5.0 https://github.com/pgvector/pgvector.git
cd pgvector
make
make install # may need sudo
```

Luego se habilita la extensión en `psql`:
```
psql

francisco=# CREATE EXTENSION vector;
CREATE EXTENSION
```
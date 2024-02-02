---
title: "Async Alembic Notes"
date: 2023-09-26T23:16:26-06:00
lastmod: 
draft: false
tags: ["alembic", "notes", "sqlalchemy", "migrations", "python"]
categories: ["tech"]
---

Notes on transitioning from a synchronous setup using fastapi, sqlalchemy, and alembic with a sqlite database to a setup that uses fastapi, sqlalchemy, and alembic, but completely async with a mysql db (using [aiomysql](https://github.com/aio-libs/aiomysql))

I will only concentrate on the alembic setup and assume a mysql user with sufficient privileges.

1. install aiomysql  
> `pip install aiomysql`

2. alembic's `env.py`  
Assuming a transition from the basic `env.py` that is generated by running `alembic init` without any template selected, the file has to be adapted to look *something* like this. All changes taken from the [alembic docs - using asyncio with alembic](https://alembic.sqlalchemy.org/en/latest/cookbook.html#using-asyncio-with-alembic)[^1] and basically affects "only `run_migrations_online` will need to be updated".

```python {hl_lines=[3, "64-98"]}
import asyncio

from sqlalchemy.ext.asyncio import async_engine_from_config

from logging.config import fileConfig

from sqlalchemy import engine_from_config
from sqlalchemy import pool

from alembic import context

# import Base from database
from src.data.database import Base

# import all Classes from the model files
from src.models import Class1, Class2

# this is the Alembic Config object, which provides
# access to the values within the .ini file in use.
config = context.config

# Interpret the config file for Python logging.
# This line sets up loggers basically.
if config.config_file_name is not None:
    fileConfig(config.config_file_name)

# add your model's MetaData object here
# for 'autogenerate' support
# from myapp import mymodel
# target_metadata = mymodel.Base.metadata
target_metadata = Base.metadata

# other values from the config, defined by the needs of env.py,
# can be acquired:
# my_important_option = config.get_main_option("my_important_option")
# ... etc.


def run_migrations_offline() -> None:
    """Run migrations in 'offline' mode.

    This configures the context with just a URL
    and not an Engine, though an Engine is acceptable
    here as well.  By skipping the Engine creation
    we don't even need a DBAPI to be available.

    Calls to context.execute() here emit the given string to the
    script output.

    """
    url = config.get_main_option("sqlalchemy.url")
    context.configure(
        url=url,
        target_metadata=target_metadata,
        literal_binds=True,
        dialect_opts={"paramstyle": "named"},
    )

    with context.begin_transaction():
        context.run_migrations()



def do_run_migrations(connection):
    context.configure(connection=connection, target_metadata=target_metadata)

    with context.begin_transaction():
        context.run_migrations()


async def run_async_migrations():
    """In this scenario we need to create an Engine
    and associate a connection with the context.

    """

    connectable = async_engine_from_config(
        config.get_section(config.config_ini_section),
        prefix="sqlalchemy.",
        poolclass=pool.NullPool,
    )

    async with connectable.connect() as connection:
        await connection.run_sync(do_run_migrations)

    await connectable.dispose()


def run_migrations_online():
    """Run migrations in 'online' mode."""

    asyncio.run(run_async_migrations())
    
if context.is_offline_mode():
    run_migrations_offline()
else:
    run_migrations_online()

```

3. alembic's `alembic.ini`  
This value also holds a variable pointing to your database and the driver it should use. Therefore you have to search for a variable called `sqlalchemy.url` around line 64 and use a connection string that reflects your chosen database type and driver:
> `sqlalchemy.url = mysql+aiomysql://<db user name>:<db user password>@<hostname>:<port>/<database>
`

3. connection string and SQLalchemy engine  
Assuming a set variable `SQLALCHEMY_DATABASE_URL` in some file like `database.py`, that references a db and driver that is not the desired mysql/async-driver combination, it is necessary to update this variable to use a mysql db and using the aiomysql driver:
> `SQLALCHEMY_DATABASE_URL = "mysql+aiomysql://<db user name>:<db user password>@<hostname>:<port>/<database>"`

Further, if you used a (sync) sqlite database, it is necessary to update the SQLalchemy engine creation to an async engine (and ommitting the recommended attribute for sqlite databases: `connect_args={"check_same_thread": False}`):
```python
from sqlalchemy.ext.asyncio import create_async_engine

engine = create_async_engine(SQLALCHEMY_DATABASE_URL)
```

4. update `models.py`  
If the tables and columns in `models.py` use some dialect specific instruction, it is necessary to update those to match, in this case, the MYSQL requirements.
e.g. sqlite does not enforce length on `STRING` columns, MYSQL interprets `STRING` as `VARCHAR` and wants a maximum length on those columns:
> `columnname = Column(String(**200**), index=True)`  
This change needs to be added, I suppose, too to already generated alembic revisions[^2].

5. setup environments for alembic  
You probably want to separate environments, speak "development" and "production" or "whatever-you-want". I'll leave the `database.py` part up to you to figure out, since it should be pretty straightforward. For alembic we have to go back to `alembic.ini` and `env.py`. There seem to be be various opinions on how to solve this and various ways to solve the distinction between environments.  
One way described in alembic's cookbook as [run multiple alembic environments from one ini file](https://alembic.sqlalchemy.org/en/latest/cookbook.html#run-multiple-alembic-environments-from-one-ini-file) is to have various alembic environments in (one) `alembic.ini`. This however still implies having database credentials in clear text in this file and having to take care at alembic runtime to specify the desired alembic environment with the `--name` flag.  
A question asked at stack overflow: [is it possible to store the alembic connect string outside of alembic ini](https://stackoverflow.com/questions/22178339/is-it-possible-to-store-the-alembic-connect-string-outside-of-alembic-ini) provides two possible solutions that don't include database credentials in clear text and pulls the variables from environment variables. The [first proposal](https://stackoverflow.com/a/27256675/14009697) seems to leverage an internal alembic API, which is not recommended to do. The [second proposal](https://stackoverflow.com/a/55190497/14009697) describes interpolation of strings in `alembic.ini` with variables pulled in `env.py` from environment. This looks good so far, but does not account for changing environment between "development" and "production".  
So I clumped together a solution that:  
    1. makes changing between "development" and "production" easy
    2. does not store database credentials in neither `env.py` nor `alembic.ini`
    3. let's me run `alembic` commands without specifying additional flags like `--name` or `--config`

    I base my solution on having the url for the development database in `alembic.ini`'s `sqlalchemy.url` and only overwrite this value, when running alembic in production mode. All variables are stored in a single place, in my case file `config.env`.

**alembic.ini**
```python
# everything untouched except sqlalchemy.url
(...)
# specify the location of your development database, in my case a sqlite db using the async aiosqlite driver
sqlalchemy.url = sqlite+aiosqlite:///src/sql_app.db

(...)
```

**env.py**  
based on the already updated version above, some changes for the connection in `offline` and `online` are necessary. This is again the complete file, updated and marked with changed/new lines.
```python {hl_lines=["18-22", "57-60", "85-100"]}
import asyncio

from sqlalchemy.ext.asyncio import async_engine_from_config

from logging.config import fileConfig

from sqlalchemy import engine_from_config
from sqlalchemy import pool

from alembic import context

# import Base from database
from src.data.database import Base

# import all Classes from the model files
from src.models import Class1, Class2

# import the config.env file which holds variables for the environment and the database url parts
from src.config import get_config

# import the actual values from config.env
envconfig = get_config()

# this is the Alembic Config object, which provides
# access to the values within the .ini file in use.
config = context.config

# Interpret the config file for Python logging.
# This line sets up loggers basically.
if config.config_file_name is not None:
    fileConfig(config.config_file_name)

# add your model's MetaData object here
# for 'autogenerate' support
# from myapp import mymodel
# target_metadata = mymodel.Base.metadata
target_metadata = Base.metadata

# other values from the config, defined by the needs of env.py,
# can be acquired:
# my_important_option = config.get_main_option("my_important_option")
# ... etc.


def run_migrations_offline() -> None:
    """Run migrations in 'offline' mode.

    This configures the context with just a URL
    and not an Engine, though an Engine is acceptable
    here as well.  By skipping the Engine creation
    we don't even need a DBAPI to be available.

    Calls to context.execute() here emit the given string to the
    script output.

    """
    if envconfig.ENVIRONMENT == "development":
        url = config.get_main_option("sqlalchemy.url")
    elif envconfig.ENVIRONMENT == "production":
        url = f"mysql+aiomysql://{envconfig.DB_USER}:{envconfig.DB_PASSWORD}@{envconfig.DB_HOST}:{envconfig.DB_PORT}/{envconfig.DB_DATABASE}"

    context.configure(
        url=url,
        target_metadata=target_metadata,
        literal_binds=True,
        dialect_opts={"paramstyle": "named"},
    )

    with context.begin_transaction():
        context.run_migrations()


def do_run_migrations(connection):
    context.configure(connection=connection, target_metadata=target_metadata)

    with context.begin_transaction():
        context.run_migrations()


async def run_async_migrations():
    """In this scenario we need to create an Engine
    and associate a connection with the context.

    """
    if envconfig.ENVIRONMENT == "development":
        connectable = async_engine_from_config(
            config.get_section(config.config_ini_section),
            prefix="sqlalchemy.",
            poolclass=pool.NullPool,
        )
    elif envconfig.ENVIRONMENT == "production":
        async_config = {
            "sqlalchemy.url": f"mysql+aiomysql://{envconfig.DB_USER}:{envconfig.DB_PASSWORD}@{envconfig.DB_HOST}:{envconfig.DB_PORT}/{envconfig.DB_DATABASE}",
            "sqlalchemy.poolclass": "NullPool",
        }
        connectable = async_engine_from_config(
            async_config,
            prefix="sqlalchemy.",
            poolclass=pool.NullPool,
        )

    async with connectable.connect() as connection:
        await connection.run_sync(do_run_migrations)

    await connectable.dispose()


def run_migrations_online():
    """Run migrations in 'online' mode."""

    asyncio.run(run_async_migrations())


if context.is_offline_mode():
    run_migrations_offline()
else:
    run_migrations_online()
```

Some explanations:  
**Lines 18-22:** I use a function to import my config from the file `config.env`. These lines here simply import the function that returns my environment variables, which I can, as can be seen further down, be used like `envconfig.<VARIABLENAME>`.  
**Lines 57-60:** These lines check, the `ENVIRONMENT` variable set in `config.env`. In "development" alembic *falls back* to the variable set in `alembic.ini` at `sqlalchemy.url`[^3]. If I set `ENVIRONMENT` in `config.env` to "production", I use an f-string to create the full connection string (all secrets/variables pulled from `config.env`)  
**Lines 85-100:** The section for "development" is pretty straightforward to set up, since it, again, *falls back* to `alembic.ini`'s `sqlalchemy.url`. The section for "production" was a bit trickier. I solved it by defining a dictionary, mimicking the structure in `alembic.ini`, that holds the same connection string as in the `offline` section. This dictionary is passed to `async_engine_from_config`, which extracts the safe connection string form it and creates an sqlalchemy async engine, which in turn is used by alembic.

6. That's all folks

I hope I captured all my modifications. 








[^1]: [Alembic's cookbook](https://alembic.sqlalchemy.org/en/latest/cookbook.html) is delicious by the way!
[^2]: to be honest, I simply scrapped all existing revisions and create a new one, that included those changes.  
[^3]: Since I use a sqlite database for development, there is no risk of exposing database credentials. For other databases it would be necessary to infer the credentials like on line 60 and completly ignore/not set `sqlalchemy.url`.
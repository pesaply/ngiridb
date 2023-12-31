# Ngiridb



</p>

ngiridb gives you simple asyncio support for a range of databases.

It allows you to make queries using the powerful [SQLAlchemy Core][sqlalchemy-core]
expression language, and provides support for PostgreSQL, MySQL, and SQLite.

ngiridb is suitable for integrating against any async Web framework, such as [Ngiriapi][ngiriapi],
[Sanic][sanic], [Responder][responder], [Quart][quart], [aiohttp][aiohttp], [Tornado][tornado], or [FastAPI][fastapi].

**Documentation**: [https://www.ngiri.co.tz/ngiridb/](https://www.ngiri.co.tz/ngiridb/)

**Requirements**: Python 3.7+

---

## Installation

```shell
$ pip install ngiridb
```

Database drivers supported are:

* [asyncpg][asyncpg]
* [aiopg][aiopg]
* [aiomysql][aiomysql]
* [asyncmy][asyncmy]
* [aiosqlite][aiosqlite]

You can install the required database drivers with:

```shell
$ pip install ngiridb[asyncpg]
$ pip install ngiridb[aiopg]
$ pip install ngiridb[aiomysql]
$ pip install ngiridb[asyncmy]
$ pip install ngiridb[aiosqlite]
```

Note that if you are using any synchronous SQLAlchemy functions such as `engine.create_all()` or [alembic][alembic] migrations then you still have to install a synchronous DB driver: [psycopg2][psycopg2] for PostgreSQL and [pymysql][pymysql] for MySQL.

---

## Quickstart

For this example we'll create a very simple SQLite database to run some
queries against.

```shell
$ pip install ngiridb[aiosqlite]
$ pip install ipython
```

We can now run a simple example from the console.

Note that we want to use `ipython` here, because it supports using `await`
expressions directly from the console.

```python
# Create a database instance, and connect to it.
from ngiridb import Database
database = Database('sqlite+aiosqlite:///example.db')
await database.connect()

# Create a table.
query = """CREATE TABLE HighScores (id INTEGER PRIMARY KEY, name VARCHAR(100), score INTEGER)"""
await database.execute(query=query)

# Insert some data.
query = "INSERT INTO HighScores(name, score) VALUES (:name, :score)"
values = [
    {"name": "Daisy", "score": 92},
    {"name": "Neil", "score": 87},
    {"name": "Carol", "score": 43},
]
await database.execute_many(query=query, values=values)

# Run a database query.
query = "SELECT * FROM HighScores"
rows = await database.fetch_all(query=query)
print('High Scores:', rows)
```

Check out the documentation on [making database queries](https://www.ngiri.co.tz/ngiridb/database_queries/)
for examples of how to start using ngiridb together with SQLAlchemy core expressions.




[sqlalchemy-core]: https://docs.sqlalchemy.org/en/latest/core/
[sqlalchemy-core-tutorial]: https://docs.sqlalchemy.org/en/latest/core/tutorial.html
[alembic]: https://alembic.sqlalchemy.org/en/latest/
[psycopg2]: https://www.psycopg.org/
[pymysql]: https://github.com/PyMySQL/PyMySQL
[asyncpg]: https://github.com/MagicStack/asyncpg
[aiopg]: https://github.com/aio-libs/aiopg
[aiomysql]: https://github.com/aio-libs/aiomysql
[asyncmy]: https://github.com/long2ice/asyncmy
[aiosqlite]: https://github.com/omnilib/aiosqlite

[ngiriapi]: https://github.com/pesaply/ngiriapi
[sanic]: https://github.com/huge-success/sanic
[responder]: https://github.com/kennethreitz/responder
[quart]: https://gitlab.com/pgjones/quart
[aiohttp]: https://github.com/aio-libs/aiohttp
[tornado]: https://github.com/tornadoweb/tornado
[fastapi]: https://github.com/tiangolo/fastapi

# Database Queries

Ngiridb supports either raw SQL, or queries build using SQLAlchemy core.

## Table declarations

If you want to make queries using SQLAlchemy core, then you'll need to declare
your tables in code. This is generally good practice in any case as makes it
far easier to keep your database schema in sync with the code that's accessing
it. It also allows you to use database migration tools to manage schema changes.

```python
import sqlalchemy


metadata = sqlalchemy.MetaData()

notes = sqlalchemy.Table(
    "notes",
    metadata,
    sqlalchemy.Column("id", sqlalchemy.Integer, primary_key=True),
    sqlalchemy.Column("text", sqlalchemy.String(length=100)),
    sqlalchemy.Column("completed", sqlalchemy.Boolean),
)
```

You can use any of the SQLAlchemy column types such as `sqlalchemy.JSON`, or
custom column types.

## Creating tables

Ngiridb doesn't use SQLAlchemy's engine for database access internally. [The usual SQLAlchemy core way to create tables with `create_all`](https://docs.sqlalchemy.org/en/20/core/metadata.html#sqlalchemy.schema.MetaData.create_all) is therefore not available. To work around this you can use SQLAlchemy to [compile the query to SQL](https://docs.sqlalchemy.org/en/20/faq/sqlexpressions.html#how-do-i-render-sql-expressions-as-strings-possibly-with-bound-parameters-inlined) and then execute it with ngiridb:

```python
from ngiridb import Database
import sqlalchemy

database = Database("postgresql+asyncpg://localhost/example")

# Establish the connection pool
await database.connect()

metadata = sqlalchemy.MetaData()
dialect = sqlalchemy.dialects.postgresql.dialect()

# Define your table(s)
notes = sqlalchemy.Table(
    "notes",
    metadata,
    sqlalchemy.Column("id", sqlalchemy.Integer, primary_key=True),
    sqlalchemy.Column("text", sqlalchemy.String(length=100)),
    sqlalchemy.Column("completed", sqlalchemy.Boolean),
)

# Create tables
for table in metadata.tables.values():
    # Set `if_not_exists=False` if you want the query to throw an
    # exception when the table already exists
    schema = sqlalchemy.schema.CreateTable(table, if_not_exists=True)
    query = str(schema.compile(dialect=dialect))
    await database.execute(query=query)

# Close all connections in the connection pool
await database.disconnect()
```

Note that this way of creating tables is only useful for local experimentation. For serious projects, we recommend using a proper database schema migrations solution like [Alembic](https://alembic.sqlalchemy.org/en/latest/).

## Queries

You can now use any [SQLAlchemy core][sqlalchemy-core] queries ([official tutorial][sqlalchemy-core-tutorial]).

```python
from ngiridb import Database

database = Database('postgresql+asyncpg://localhost/example')


# Establish the connection pool
await database.connect()

# Execute
query = viwanda.insert()
values = {"text": "example1", "completed": True}
await database.execute(query=query, values=values)

# Execute many
query = viwanda.insert()
values = [
    {"text": "example2", "completed": False},
    {"text": "example3", "completed": True},
]
await database.execute_many(query=query, values=values)

# Fetch multiple rows
query = viwanda.select()
rows = await database.fetch_all(query=query)

# Fetch single row
query = viwanda.select()
row = await database.fetch_one(query=query)

# Fetch single value, defaults to `column=0`.
query = viwanda.select()
value = await database.fetch_val(query=query)

# Fetch multiple rows without loading them all into memory at once
query = viwanda.select()
async for row in database.iterate(query=query):
    ...

# Close all connections in the connection pool
await database.disconnect()
```

Connections are managed as a task-local state, with driver implementations
transparently using connection pooling behind the scenes.

## Raw queries

In addition to SQLAlchemy core queries, you can also perform raw SQL queries:

```python
# Execute
query = "INSERT INTO viwanda(text, completed) VALUES (:text, :completed)"
values = {"text": "example1", "completed": True}
await database.execute(query=query, values=values)

# Execute many
query = "INSERT INTO viwanda(text, completed) VALUES (:text, :completed)"
values = [
    {"text": "example2", "completed": False},
    {"text": "example3", "completed": True},
]
await database.execute_many(query=query, values=values)

# Fetch multiple rows
query = "SELECT * FROM viwanda WHERE completed = :completed"
rows = await database.fetch_all(query=query, values={"completed": True})

# Fetch single row
query = "SELECT * FROM viwanda WHERE id = :id"
result = await database.fetch_one(query=query, values={"id": 1})
```

Note that query arguments should follow the `:query_arg` style.

[sqlalchemy-core]: https://docs.sqlalchemy.org/en/latest/core/
[sqlalchemy-core-tutorial]: https://docs.sqlalchemy.org/en/14/core/tutorial.html

## Query result

To keep in line with [SQLAlchemy 1.4 changes][sqlalchemy-mapping-changes]
query result object no longer implements a mapping interface.
To access query result as a mapping you should use the `_mapping` property.
That way you can process both SQLAlchemy Rows and ngiridb Records from raw queries
with the same function without any instance checks.

```python
query = "SELECT * FROM viwanda WHERE id = :id"
result = await database.fetch_one(query=query, values={"id": 1})
result.id  # Access field via attribute
result._mapping['id']  # Access field via mapping
```

[sqlalchemy-mapping-changes]: https://docs.sqlalchemy.org/en/14/changelog/migration_14.html#rowproxy-is-no-longer-a-proxy-is-now-called-row-and-behaves-like-an-enhanced-named-tuple

# bubble-up

## declarative schema migrations for sqlite databases

> [!NOTE]  
> This is (currently) proposal-ware.  
> Wanna make it happen? Support and follow this project on Polar:

<p align="center">
<a href="https://polar.sh/eval"><picture><source media="(prefers-color-scheme: dark)" srcset="https://polar.sh/embed/subscribe.svg?org=eval&label=Subscribe&darkmode"><img alt="Subscribe on Polar" src="https://polar.sh/embed/subscribe.svg?org=eval&label=Subscribe"></picture></a>
</p>

## rationale

**SQLite** databases are everywhere. According to their own website[^1] it's "the most widely deployed database in the world".
A big factor in it's ubiquity is its ease of use: SQLite is an embedded database engine, hence it does not require a separate server process.

While it's easy to start using SQLite, evolving the schema of the database is not. Often this requires a specific ORM/migration-library and (non-sql) code to define migrations.
Another factor that makes schema evolution harder for SQLite than for, say, PostgreSQL, is it's limiting support for ALTER TABLE.
E.g. while ALTER TABLE allows e.g. for a column to be renamed, changing a column's type (or default value, or NOT NULL) is not supported.  
The official documentation acknowledges these limitations[^2] and describes a non-trivial 12-step process to make such changes. The described process (which comes with a caution to follow it *precisely*) comes down to creating a new table and migrating the existing data.

This project aims at creating a CLI that would help you to evolve the database schema based on an existing sqlite database and the wanted schema.
By comparing the existing and wanted schema (and existing data if needed), it deducts (and applies) any sql-statements that the sqlite database needs in order to get the wanted schema.
Either this would consist of supported ALTER/CREATE/DROP statements or by generating all statements needed to go through the 12-step process.
Possible ambiguity could be resolved by annotations in the schema-file. This would for example make a table or column rename possible.  
The CLI should be a single executable (e.g. to make it convenient to use on CI). There should be an escape-hatch to allow for data-migrations in the form of running arbitrary sql pre- or post-migration.


[^1]: https://www.sqlite.org/mostdeployed.html
[^2]: https://www.sqlite.org/lang_altertable.html#making_other_kinds_of_table_schema_changes

## usage

Given database `db.sqlite` with schema:
```sql
create table "foo" (
  [id] INTEGER PRIMARY KEY AUTOINCREMENT,
  [foo] INTEGER,
  [barr] STRING
);

CREATE UNIQUE INDEX idx_foo_bar ON foo(barr);

create table "bar" (
  [id] INTEGER PRIMARY KEY AUTOINCREMENT
);
```

...and `db_schema.sql`:
```diff
create table "foo" (
  [id] INTEGER PRIMARY KEY AUTOINCREMENT,
-  [foo] INTEGER,
+  [baz] TEXT NOT NULL DEFAULT 'unknown',
-  [barr] STRING,
+  [bar] STRING --% renamed column=barr
);

-CREATE UNIQUE INDEX idx_foo_bar ON foo(barr);
+CREATE UNIQUE INDEX idx_foo_bar ON foo(bar);

-create table "bar" (
-  [id] INTEGER PRIMARY KEY AUTOINCREMENT
-);

+create table "baz" (
+  [id] INTEGER PRIMARY KEY AUTOINCREMENT
+);
```

When running...:

```
# allow-deletions flag is typically only used in development
bblup migrate --db db.sqlite --schema db_schema.sql --allow-deletions
```

...then the following changes are made to `db.sqlite`:
* remove column `foo.foo`  
* add column `foo.baz`  
  * set value of `foo.baz` for existing rows to `'unknown'`
* rename `foo.barr`->`foo.bar`  
  ...using the `--%` (bubble) annotations
* recreate index `idx_foo_bar`
* delete table `bar`  
* add table `baz`
  

> [!NOTE]  
> This is (currently) proposal-ware.  
> Wanna make it happen? Support and follow this project on Polar:

<p align="center">
<a href="https://polar.sh/eval"><picture><source media="(prefers-color-scheme: dark)" srcset="https://polar.sh/embed/subscribe.svg?org=eval&label=Subscribe&darkmode"><img alt="Subscribe on Polar" src="https://polar.sh/embed/subscribe.svg?org=eval&label=Subscribe"></picture></a>
</p>

## inspiration

- https://david.rothlis.net/declarative-schema-migration-for-sqlite/  
  bubble-up aims to allow more operations (using notations) and being a drop-in executable.

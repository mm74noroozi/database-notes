# database-notes
## GENERAL
general subjects

## PostgreSQL
postgresql related topics
### geometery datatypes and functions
https://www.postgresql.org/docs/8.2/functions-geometry.html
### hash and crypto
install
```sql
CREATE EXTENSION IF NOT EXISTS pgcrypto;
```
run
```sql
SELECT DIGEST('hello_world', 'sha256') AS hashed_value;
```
### search_path
In PostgreSQL, the search_path is a configuration parameter that determines the order in which schemas are searched when an unqualified table or schema name is referenced in a SQL statement. It essentially defines the set of schemas that PostgreSQL should look into to resolve object references.
### counting users by name
```sql
create table users(
    id serial,
    name varchar(8) unique,
 count int
 );
```
then
```
insert into users(name, count)
 values('Joe', 1)
 on conflict (name) do update set count = users.count + 1
```
### upsert source to target
```sql
MERGE INTO targetTable t
USING sourceTable s
    ON t.PKID = s.PKID
WHEN MATCHED AND NOT EXISTS (
        SELECT s.ColumnA, s.ColumnB, s.ColumnC
        INTERSECT
        SELECT t.ColumnA, t.ColumnB, s.ColumnC
        )
    THEN UPDATE SET
        t.ColumnA = s.ColumnA
        ,t.ColumnB = s.ColumnB
        ,t.ColumnC = s.ColumnC
WHEN NOT MATCHED BY TARGET
    THEN INSERT (PKID, ColumnA, ColumnB, ColumnC)
    VALUES (s.PKID, s.ColumnA, s.ColumnB, s.ColumnC)
WHEN NOT MATCHED BY SOURCE
    THEN DELETE;
```
## SQLSERVER
sqlserver related topics

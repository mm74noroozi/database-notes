# database-notes
## GENERAL
general subjects

## PostgreSQL
postgresql related topics
### geometery datatypes and functions
https://www.postgresql.org/docs/8.2/functions-geometry.html
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
## SQLSERVER
sqlserver related topics

# database-notes
## GENERAL
### logical order
1. FROM: A Cartesian product (cross join) is performed between the first two tables in the FROM clause, and as a result, virtual table VT1 is generated.
2. ON: The ON filter is applied to VT1. Only rows for which the is TRUE are inserted to VT2.
3. OUTER (join): If an OUTER JOIN is specified (as opposed to a CROSS JOIN or an INNER JOIN), rows from the
 preserved table or tables for which a match was not found are added to the rows from VT2 as outer rows,
 generating VT3. If more than two tables appear in the FROM clause, steps 1 through 3 are applied repeatedly
 between the result of the last join and the next table in the FROM clause until all tables are processed.
4. WHERE: The WHERE filter is applied to VT3. Only rows for which the is TRUE are inserted to VT4.
 GROUP BY: The rows from VT4 are arranged in groups based on the column list specified in the GROUP BY
 clause. VT5 is generated.
5. CUBE | ROLLUP: Supergroups (groups of groups) are added to the rows from VT5, generating VT6.
6. HAVING: The HAVING filter is applied to VT6. Only groups for which the is TRUE are inserted to VT7.
7. SELECT: The SELECT list is processed, generating VT8.
8. DISTINCT: Duplicate rows are removed from VT8. VT9 is generated.
9. ORDER BY: The rows from VT9 are sorted according to the column list specified in the ORDER BY clause. A
 cursor is generated (VC10).
10. TOP: The specified number or percentage of rows is selected from the beginning of VC10. Table VT11 is
 generated and returned to the caller. LIMIT has the same functionality as TOP in some SQL dialects such as
 Postgres and Netezza
### triggers
### concatination
all except sqlserver
```sql
 SELECT 'Hello' || 'World' || '!'; --returns HelloWorld!
```
all
```sql
 SELECT CONCAT('Hello', 'World', '!'); --returns 'HelloWorld!'
```
### string aggregation
- postgres
```sql
SELECT ColumnA
     , STRING_AGG(ColumnB, ',' ORDER BY ColumnB) AS ColumnBs
  FROM TableName
 GROUP BY ColumnA
 ORDER BY ColumnA;
```
- sql server
```sql
SELECT ColumnA
     , STRING_AGG(ColumnB, ',') WITHIN GROUP (ORDER BY ColumnB) AS ColumnBs
  FROM TableName
 GROUP BY ColumnA
 ORDER BY ColumnA;
```
- mysql
```sql
SELECT ColumnA
 , GROUP_CONCAT(ColumnB ORDER BY ColumnB SEPARATOR ',') AS ColumnBs
 FROM TableName
 GROUP BY ColumnA
 ORDER BY ColumnA;
```
### STRING_SPLIT
```sql
 SELECT value FROM STRING_SPLIT('Lorem ipsum dolor sit amet.', ' ');
```
### Clustered, Unique, and Sorted Indexes
```sql
 CREATE CLUSTERED INDEX ix_clust_employee_id ON Employees(EmployeeId, Email);
 CREATE UNIQUE INDEX uq_customers_email ON Customers(Email);
 CREATE INDEX ix_eid_desc ON Customers(EmployeeID Desc); 
```
###  Delete All But Last Record (1 to Many Table)
```sql
WITH cte AS (
 SELECT ProjectID,
 ROW_NUMBER() OVER (PARTITION BY ProjectID ORDER BY InsertDate DESC) AS rn
 FROM ProjectNotes
 )
 DELETE FROM cte WHERE rn > 1;
```
### cross apply and outer apply
- The CROSS APPLY operator is used to invoke a table-valued function for each row returned by the outer table expression.
- The OUTER APPLY operator is similar to CROSS APPLY, but it returns all rows from the outer table expression, even if there is no match with the table-valued function.
```sql
CREATE FUNCTION dbo.fn_GetAllEmployeeOfADepartment (@DeptID AS int)
RETURNS TABLE
AS
RETURN
(
SELECT
*
FROM Employee E
WHERE E.DepartmentID = @DeptID
)
GO
```
and
```sql
SELECT
*
FROM Department D
CROSS APPLY dbo.fn_GetAllEmployeeOfADepartment(D.DepartmentID)
GO
```
or
```sql
SELECT
*
FROM Department D
OUTER APPLY dbo.fn_GetAllEmployeeOfADepartment(D.DepartmentID)
GO
```
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
### DEFERRABLE INITIALLY DEFERRED
```sql
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  agency_id INTEGER NOT NULL REFERENCES agencies(id) DEFERRABLE INITIALLY DEFERRED
);
```
1. DEFERRABLE: This clause indicates that a constraint (in this case, a foreign key constraint) can be deferred. When a constraint is deferrable, it means that the database can check the constraint at the end of the transaction rather than immediately when the data is modified. This can be useful in certain situations where you need to temporarily violate a constraint within a transaction but ensure that it is satisfied by the end of the transaction.

2. INITIALLY DEFERRED: This further refines the deferrable constraint, specifying that the constraint is initially checked at the end of the transaction. In other words, when a transaction begins, the database allows the foreign key constraint to be violated temporarily, and the actual check is deferred until the end of the transaction.
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
### regr_slope
```sql
SELECT regr_slope(sales, advertising) AS slope
FROM sales_data;
```
This will return the slope of the regression line, indicating the change in sales for a one-unit change in advertising expenses.
## SQLSERVER
### Partial or Filtered Index
```sql
CREATE INDEX Started_Orders
 ON orders(product_id)
 WHERE order_state_id = 1;
```
###  Create and call a stored procedure
```sql
-- Define a name and parameters
 CREATE PROCEDURE Northwind.getEmployee
    @LastName nvarchar(50),  
    @FirstName nvarchar(50)  
AS  -- Define the query to be run
 SELECT FirstName, LastName, Department  
FROM Northwind.vEmployeeDepartment
 WHERE FirstName = @FirstName AND LastName = @LastName  
AND EndDate IS NULL; 
```
Calling the procedure:
```sql
EXECUTE Northwind.getEmployee N'Ackerman', N'Pilar';-- Or  
EXEC Northwind.getEmployee @LastName = N'Ackerman', @FirstName = N'Pilar';  
GO  -- Or  
EXECUTE Northwind.getEmployee @FirstName = N'Pilar', @LastName = N'Ackerman';  
GO 
```

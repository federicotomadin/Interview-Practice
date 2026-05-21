
## DDL - Definition Data Language
- Create
- Alter
- Truncate
- Comment
- Rename
- Drop

## DML - Definition Manipulation Language
- Insert
- Select
- Delete
- Update

## DCL - Data Control Language
- Grant - Brinda privilegios
- Revoke - Quita privilegios

## TCL - Transaction Control Language
- Commit - Guarda el trabajo hecho
- Rollback - Deshace la modificación del último commit
- Set -> Establece aislamiento de transacción
- Savepoint - Salva un punto al que se puede retroceder en la transacción

# SQL Commands

### JOIN
- Inner join
- Left join
- Right join
- Full join
- Self join

### AGGREGATION
- Sum
- Avg
- Count
- Min
- Max

### GROUP BY
- Group by
- Having

### ALIAS
- As

### ORDER BY
- Order by asc
- Order by desc

### WHERE
- Exist
- Like
- In
- Not
- Or , And
- All
- Between
- Any

## Truncate vs Delete

1. **Efficiency:**
    - `TRUNCATE` is generally more efficient than `DELETE` for large datasets because it doesn't generate individual row deletion operations and doesn't log each deleted row.
    - `DELETE` can be less efficient because it involves logging each row deletion and may have additional overhead.
2. **Rollback:**
    - `TRUNCATE` cannot be rolled back. Once a `TRUNCATE` operation is executed, it cannot be undone.
    - `DELETE` can be rolled back within a transaction, allowing you to undo the changes made by the `DELETE` statement.
3. **Identity Columns:**
    - The behavior regarding identity columns (auto-incrementing columns) may vary between database systems.
    - Some databases automatically reset identity columns to their initial seed value after a `TRUNCATE` operation, while others may not.
    - Deleting rows using `DELETE` does not necessarily reset identity columns unless explicitly specified.
4. **DML and DDL:**
    - `TRUNCATE` is a DDL (Data Definition Language) command.
    - `DELETE` is a DML (Data Manipulation Language) command.
5. **Speed:**
    - `TRUNCATE` is much faster than `DELETE`, as it doesn't scan every record before removing it.
    - `TRUNCATE` removes the data by deallocating the data pages used to store the table's data, and only the page deallocations are recorded in the transaction log.
    - `DELETE` removes rows one at a time and records an entry in the transaction log for each deleted row.
6. **Referential Integrity:** 
    - `TRUNCATE` cannot be used if other tables reference the table containing the data to be truncated using foreign key constraints.
    - `DELETE` can be used regardless of whether or not other tables reference the table containing the data to be deleted.
7. **Conditions:**
    - `TRUNCATE` cannot be used with a `WHERE` clause.
    - `DELETE` can be used with or without a `WHERE` clause.

## Store Procedure vs Function

- Function
    - Must have at least one parameter
    - MUST return a value
    - Cannot alter the data they receive as parameters (the arguments)
    - Generally cannot contain transaction control statements, and they are meant for read-only operations.
    - Called directly as part of a SELECT statement or used in an expression.
    - Cannot call another function or stored procedure.

- Store Procedure
    - Stored procs do not have to have a parameter,
    - Can change database objects
    - Do not have to return a value
    - Can contain transaction control statements (e.g., COMMIT, ROLLBACK).
    - Allows the use of TRY...CATCH blocks for error handling.
    - Called using the EXECUTE or EXEC keyword.
    - Can call another stored procedure.

## Joins in SQL

- Inner join
- Outer join
    - Left Join  (Left outer join)
    - Right Join (Right outer join)
    - Full Join (Full outer join)
- Self join
    - A self join is a specific case of a SQL join where a table is joined with itself. In other words, you use the same table on both sides of the join operation. This is useful when you want to compare rows within the same table or when dealing with hierarchical data stored in a single table.

## Distinct

- The `DISTINCT` keyword in SQL is used in a `SELECT` statement to eliminate duplicate rows from the result set. When you use `DISTINCT`, the database engine considers only unique values for the specified columns, and it returns a result set with those unique values.

## Pros of SQL Databases

- *Strong consistency and data integrity.*
- *Well-suited for complex queries and relationships between data.*
- *Mature technology with a wide range of tools and support.*
- *ACID compliance ensures reliable transactions.*

## Cons of SQL Databases

- *Lack of flexibility in handling unstructured or semi-structured data.*
- *Vertical scalability can be costly.*
- *Not ideal for rapidly evolving schemas or applications requiring high write throughput.*

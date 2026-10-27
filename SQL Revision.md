# SQL Complete Revision for Backend Developers

We'll use MySQL syntax for most examples, because it's a practical starting point for  backend development. I'll point out PostgreSQL differences where useful.

The goal isn't just to memorize SQL commands. You should understand how to design a database, query data, maintain relationships, and work with SQL through Java applications.

## 1. SQL fundamentals

SQL stands for Structured Query Language. It lets you communicate with a relational database.

For example, imagine you're developing a university management system.

Database: `university_db`

Contains tables and relationships.

Table: `students`

Stores student records in rows and columns.

Row: One student

Example: ID 1, Ahmed, age 21.

### Main categories of SQL commands

| Category | Purpose                   | Commands                              |
| -------- | ------------------------- | ------------------------------------- |
| DDL      | Define database structure | `CREATE`, `ALTER`, `DROP`, `TRUNCATE` |
| DML      | Insert or modify data     | `INSERT`, `UPDATE`, `DELETE`          |
| DQL      | Query data                | `SELECT`                              |
| DCL      | Manage permissions        | `GRANT`, `REVOKE`                     |
| TCL      | Manage transactions       | `COMMIT`, `ROLLBACK`, `SAVEPOINT`     |

## 2. Database and table creation (DDL)

### CREATE DATABASE

```
CREATE DATABASE university_db;

USE university_db;
```

- `CREATE DATABASE` creates a database.
- `USE` selects the database in MySQL.

### CREATE TABLE

```
CREATE TABLE students (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(150) UNIQUE,
    age INT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

Explanation:

| Command or constraint | Meaning                                    |
| --------------------- | ------------------------------------------ |
| `BIGINT`              | Large integer                              |
| `VARCHAR(100)`        | Variable-length text, up to 100 characters |
| `PRIMARY KEY`         | Uniquely identifies each row               |
| `AUTO_INCREMENT`      | MySQL generates an ID automatically        |
| `NOT NULL`            | Value is required                          |
| `UNIQUE`              | Duplicate non-NULL values are prohibited   |
| `DEFAULT`             | Supplies a value when one isn't specified  |
| `CURRENT_TIMESTAMP`   | Uses the current date and time             |

Important: A primary key must be unique and non-NULL. A table can have only one primary key constraint, although it can contain multiple columns as a composite key.

### Common SQL data types

| Type            | Example                 | Typical use                                               |
| --------------- | ----------------------- | --------------------------------------------------------- |
| `INT`           | `25`                    | Age, quantity                                             |
| `BIGINT`        | `10000000000`           | Large IDs                                                 |
| `DECIMAL(10,2)` | `1250.75`               | Money and precise decimal values                          |
| `VARCHAR(255)`  | `'Ahmed Fekry'`         | Names and emails                                          |
| `TEXT`          | Long description        | Articles, comments                                        |
| `DATE`          | `'2026-10-28'`          | Birth date                                                |
| `DATETIME`      | `'2026-10-28 12:30:00'` | Date and time                                             |
| `TIMESTAMP`     | Current timestamp       | Audit timestamps                                          |
| `BOOLEAN`       | `TRUE`                  | Logical values; MySQL represents these using `TINYINT(1)` |

Use `DECIMAL` rather than floating-point types when exact decimal precision matters, such as financial calculations.

## 3. ALTER TABLE — Change the structure

Assume the `students` table already exists.

### Add a column

```
ALTER TABLE students
ADD COLUMN address VARCHAR(200);
```

### Rename a column

```
ALTER TABLE students
RENAME COLUMN name TO full_name;
```

### Modify a column's type

```
ALTER TABLE students
MODIFY COLUMN full_name VARCHAR(150) NOT NULL;
```

This is MySQL syntax. PostgreSQL uses `ALTER COLUMN ... TYPE`.

### Remove a column

```
ALTER TABLE students
DROP COLUMN address;
```

### Rename a table

```
RENAME TABLE students TO university_students;
```

### Add a constraint

```
ALTER TABLE students
ADD CONSTRAINT uq_students_email UNIQUE (email);
```

Use `ALTER TABLE` when you need to change the schema without recreating the whole table. Be careful: schema changes can affect existing application code and data.

## 4. INSERT — Add data

### Insert one row

```
INSERT INTO students (name, email, age)
VALUES ('Ahmed', 'ahmed@example.com', 21);
```

The database generates `id` and `created_at` automatically.

### Insert multiple rows

```
INSERT INTO students (name, email, age)
VALUES
    ('Mona', 'mona@example.com', 22),
    ('Omar', 'omar@example.com', 20),
    ('Sara', 'sara@example.com', 21);
```

### Insert only selected columns

```
INSERT INTO students (name, age)
VALUES ('Ali', 23);
```

The omitted columns receive their default values or `NULL` if permitted. If `email` is unique but nullable, MySQL permits multiple `NULL` values in that column.

Backend rule: When inserting through Java, use prepared statements or parameterized ORM queries. Never build SQL by concatenating untrusted user input.

## 5. SELECT — Retrieve data

This is one of the most important SQL commands for backend developers.

### Retrieve all columns

```
SELECT *
FROM students;
```

`*` selects all columns. In production application code, explicitly listing required columns is often preferable.

### Select specific columns

```
SELECT id, name, email
FROM students;
```

### Filter with WHERE

```
SELECT *
FROM students
WHERE age >= 21;
```

### Comparison operators

| Operator     | Meaning               |
| ------------ | --------------------- |
| `=`          | Equal                 |
| `<>` or `!=` | Not equal             |
| `>`          | Greater than          |
| `<`          | Less than             |
| `>=`         | Greater than or equal |
| `<=`         | Less than or equal    |

### AND, OR, NOT

```
SELECT *
FROM students
WHERE age >= 20 AND age <= 22;
SELECT *
FROM students
WHERE age = 20 OR age = 22;
SELECT *
FROM students
WHERE NOT age = 20;
```

Use parentheses when mixing `AND` and `OR` to make your intended logic explicit.

### BETWEEN

```
SELECT *
FROM students
WHERE age BETWEEN 20 AND 22;
```

`BETWEEN` includes both boundaries.

### IN

```
SELECT *
FROM students
WHERE age IN (20, 22, 23);
```

Useful when matching a column against multiple possible values.

### LIKE — Search text patterns

```
SELECT *
FROM students
WHERE name LIKE 'A%';
```

| Pattern   | Meaning                             |
| --------- | ----------------------------------- |
| `'A%'`    | Starts with A                       |
| `'%a'`    | Ends with a                         |
| `'%med%'` | Contains med                        |
| `'_li'`   | Any single character followed by li |

`%` represents zero or more characters; `_` represents exactly one character. Case sensitivity depends on the database and collation.

### NULL handling

```
SELECT *
FROM students
WHERE email IS NULL;
SELECT *
FROM students
WHERE email IS NOT NULL;
```

Do not write `email = NULL`. SQL uses `IS NULL` and `IS NOT NULL` because `NULL` represents missing or unknown information.

### DISTINCT — Remove duplicates

```
SELECT DISTINCT age
FROM students;
```

### ORDER BY — Sort results

```
SELECT *
FROM students
ORDER BY age ASC;
SELECT *
FROM students
ORDER BY age DESC, name ASC;
```

- `ASC`: ascending, the default.
- `DESC`: descending.
- Multiple columns let you define tie-breaking order.

### LIMIT and OFFSET — Pagination

```
SELECT *
FROM students
ORDER BY id
LIMIT 10 OFFSET 20;
```

This returns up to 10 rows after skipping the first 20. This is commonly used for API pagination. Always use a deterministic `ORDER BY` for predictable results.

## 6. UPDATE and DELETE — Modify existing data

### UPDATE

```
UPDATE students
SET age = 22
WHERE id = 1;
```

Update multiple columns:

```
UPDATE students
SET age = 23,
    name = 'Ahmed Fekry'
WHERE id = 1;
```

Critical: If you omit `WHERE`, every row in the table will be updated.

### DELETE

```
DELETE FROM students
WHERE id = 1;
```

Deletes the matching student row.

```
DELETE FROM students;
```

Deletes every row while keeping the table structure.

### DELETE vs TRUNCATE vs DROP

| Command          | Result                                             |
| ---------------- | -------------------------------------------------- |
| `DELETE`         | Removes selected rows; supports `WHERE`            |
| `TRUNCATE TABLE` | Removes all rows, usually with faster bulk cleanup |
| `DROP TABLE`     | Removes the table itself                           |

`TRUNCATE` and `DROP` have different transaction and rollback behavior depending on the database. In MySQL, they cause implicit commits in common configurations, so don't assume you can roll them back.

## 7. Aggregate functions

Aggregate functions calculate results from multiple rows.

| Function  | Purpose                       |
| --------- | ----------------------------- |
| `COUNT()` | Count rows or non-NULL values |
| `SUM()`   | Calculate a total             |
| `AVG()`   | Calculate an average          |
| `MIN()`   | Find the minimum              |
| `MAX()`   | Find the maximum              |

Examples:

```
SELECT COUNT(*) AS total_students
FROM students;
SELECT AVG(age) AS average_age,
       MIN(age) AS youngest,
       MAX(age) AS oldest
FROM students;
```

`COUNT(*)` counts rows. `COUNT(email)` counts only rows where `email` is not `NULL`.

### GROUP BY

Suppose you want to count students by age:

```
SELECT age, COUNT(*) AS student_count
FROM students
GROUP BY age;
```

`GROUP BY` groups rows sharing the same value so aggregate functions can calculate results for each group.

### HAVING

Use `HAVING` to filter groups after aggregation.

```
SELECT age, COUNT(*) AS student_count
FROM students
GROUP BY age
HAVING COUNT(*) > 1;
```

This returns ages shared by more than one student.

### WHERE vs HAVING

- `WHERE` filters individual rows before grouping.
- `HAVING` filters groups after grouping.

```
SELECT age, COUNT(*) AS student_count
FROM students
WHERE age >= 20
GROUP BY age
HAVING COUNT(*) >= 2;
```

This first excludes students younger than 20, groups the remaining rows by age, and keeps groups containing at least two students.

## 8. Relationships and database constraints

Real backend databases contain related tables, not just one table.

Let's create a `departments` table and connect students to departments.

```
CREATE TABLE departments (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL UNIQUE
);

CREATE TABLE students (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(150) UNIQUE,
    department_id BIGINT,
    FOREIGN KEY (department_id)
        REFERENCES departments(id)
);
```

The `department_id` column links a student to a department.

### Primary key vs foreign key

- Primary key (PK): Uniquely identifies a record.
- Foreign key (FK): References a key in another table and enforces referential integrity.

### Types of relationships

One-to-one (1:1)

One user has one profile. The foreign key usually needs a `UNIQUE` constraint to enforce this relationship.

One-to-many (1:N)

One department has many students. Store `department_id` in the `students` table.

Many-to-many (M:N)

One student takes many courses, and one course has many students. Use a junction table such as `student_courses`.

### Many-to-many example

```
CREATE TABLE courses (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    title VARCHAR(150) NOT NULL
);

CREATE TABLE student_courses (
    student_id BIGINT NOT NULL,
    course_id BIGINT NOT NULL,
    enrollment_date DATE,
    PRIMARY KEY (student_id, course_id),
    FOREIGN KEY (student_id) REFERENCES students(id),
    FOREIGN KEY (course_id) REFERENCES courses(id)
);
```

The composite primary key prevents the same student-course combination from appearing twice.

### Other important constraints

```
CREATE TABLE products (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    price DECIMAL(10,2) NOT NULL CHECK (price >= 0),
    stock INT NOT NULL DEFAULT 0,
    sku VARCHAR(50) UNIQUE
);
```

- `CHECK`: Enforces a condition on a value.
- `DEFAULT`: Supplies a default value.
- `UNIQUE`: Prevents duplicate values.
- `NOT NULL`: Requires a value.

MySQL enforces `CHECK` constraints in modern versions (8.0.16 and later).

## 9. SQL JOINs

JOINs are essential for Java backend interviews. They let you retrieve related data from multiple tables.

Assume these records exist:

```
departments
```

| id   | name             |
| ---- | ---------------- |
| 1    | Computer Science |
| 2    | Business         |
| 3    | Engineering      |

```
students
```

| id   | name  | department_id |
| ---- | ----- | ------------- |
| 1    | Ahmed | 1             |
| 2    | Mona  | 2             |
| 3    | Omar  | 1             |
| 4    | Sara  | NULL          |

### INNER JOIN

Returns only records that match in both tables.

```
SELECT s.name, d.name AS department
FROM students s
INNER JOIN departments d
    ON s.department_id = d.id;
```

Result: Ahmed–Computer Science, Mona–Business, Omar–Computer Science. Sara is excluded because she has no department.

### LEFT JOIN

Returns every row from the left table, including unmatched rows.

```
SELECT s.name, d.name AS department
FROM students s
LEFT JOIN departments d
    ON s.department_id = d.id;
```

Sara appears with `NULL` as her department.

### RIGHT JOIN

Returns every row from the right table, including departments with no students.

```
SELECT s.name, d.name AS department
FROM students s
RIGHT JOIN departments d
    ON s.department_id = d.id;
```

### CROSS JOIN

Returns every possible combination of rows from both tables.

```
SELECT s.name, d.name
FROM students s
CROSS JOIN departments d;
```

If there are 4 students and 3 departments, this produces 12 combinations.

### SELF JOIN

A table can join to itself, for example to represent employees and their managers.

```
SELECT e.name AS employee,
       m.name AS manager
FROM employees e
LEFT JOIN employees m
    ON e.manager_id = m.id;
```

Here, `e` and `m` are aliases for two logical roles played by the same table.

### JOIN cheat sheet

| JOIN         | Result                                 |
| ------------ | -------------------------------------- |
| `INNER JOIN` | Only matching rows                     |
| `LEFT JOIN`  | All left rows plus matching right rows |
| `RIGHT JOIN` | All right rows plus matching left rows |
| `CROSS JOIN` | All combinations                       |
| `SELF JOIN`  | A table joined to itself               |

A `FULL OUTER JOIN` returns unmatched rows from both sides as well as matching rows. MySQL does not support it directly; it can be emulated using a combination of left and right joins with duplicate handling.

## 10. Subqueries

A subquery is a query nested inside another query.

### Find students older than the average age

```
SELECT name, age
FROM students
WHERE age > (
    SELECT AVG(age)
    FROM students
);
```

The inner query calculates the average age. The outer query returns students above that average.

### IN with a subquery

```
SELECT name
FROM students
WHERE department_id IN (
    SELECT id
    FROM departments
    WHERE name = 'Computer Science'
);
```

### EXISTS

```
SELECT d.name
FROM departments d
WHERE EXISTS (
    SELECT 1
    FROM students s
    WHERE s.department_id = d.id
);
```

Returns departments that have at least one student.

Interview note: `EXISTS` checks whether matching rows exist. It is particularly useful for existence checks and correlated subqueries. The optimizer may transform equivalent `IN` and `EXISTS` queries similarly, so neither is universally faster.

## 11. CASE, COALESCE and useful expressions

### CASE — Conditional logic

```
SELECT name, age,
    CASE
        WHEN age < 18 THEN 'Minor'
        WHEN age < 22 THEN 'Young Adult'
        ELSE 'Adult'
    END AS age_group
FROM students;
```

`CASE` is SQL's conditional expression, similar in purpose to an `if/else` expression.

### COALESCE — Handle NULL values

```
SELECT name,
       COALESCE(email, 'No email') AS email
FROM students;
```

Returns the first non-NULL expression.

### IFNULL in MySQL

```
SELECT IFNULL(email, 'No email') AS email
FROM students;
```

`IFNULL` is MySQL-specific; `COALESCE` is more portable across database systems.

### Aliases

```
SELECT name AS student_name
FROM students;
```

`AS` assigns a temporary name to a column or table in a query. It doesn't rename the actual database column.

## 12. Set operations

Set operations combine results from multiple queries.

### UNION

```
SELECT email FROM students
UNION
SELECT email FROM teachers;
```

`UNION` combines results and removes duplicates.

### UNION ALL

```
SELECT email FROM students
UNION ALL
SELECT email FROM teachers;
```

`UNION ALL` retains duplicates and is usually preferable when duplicate removal isn't required.

The queries must return the same number of columns with compatible data types.

## 13. Common Table Expressions (CTEs)

A CTE is a named result set available within a single SQL statement.

```
WITH department_counts AS (
    SELECT department_id, COUNT(*) AS total
    FROM students
    GROUP BY department_id
)
SELECT department_id, total
FROM department_counts
WHERE total >= 2;
```

CTEs improve readability when a query has multiple steps. Recursive CTEs can also represent hierarchies, such as organizational charts. A CTE is not automatically faster than an equivalent subquery; performance depends on the database optimizer and query.

## 14. Views

A view is a saved query that can be queried like a table.

```
CREATE VIEW student_departments AS
SELECT s.id, s.name, d.name AS department
FROM students s
LEFT JOIN departments d
    ON s.department_id = d.id;
```

Use it:

```
SELECT *
FROM student_departments;
```

Remove it:

```
DROP VIEW student_departments;
```

A regular view typically stores the query definition, not a separate copy of its result. Materialized views, supported by some databases, store results separately.

## 15. Indexes and query performance

An index helps the database locate records without necessarily scanning the entire table.

```
CREATE INDEX idx_students_name
ON students(name);
```

A composite index:

```
CREATE INDEX idx_students_department_age
ON students(department_id, age);
```

Remove an index in MySQL:

```
DROP INDEX idx_students_name ON students;
```

### Why indexes matter

Imagine an API that searches for an order by `customer_id`. If the table contains millions of orders, an appropriate index can greatly improve lookup performance.

But indexes also have costs:

- They consume storage.
- They add overhead to inserts, updates and deletes.
- Too many indexes can hurt write performance.
- Composite index column order matters.

For the index `(department_id, age)`, queries filtering by `department_id` can benefit directly. Queries filtering only by `age` may not benefit as effectively from that index.

### EXPLAIN

```
EXPLAIN
SELECT *
FROM students
WHERE department_id = 1;
```

`EXPLAIN` helps you inspect the database's execution plan and determine whether it uses an index or scans a table.

For performance tuning, learn to inspect actual query plans, identify slow queries, and index based on real query patterns rather than adding indexes everywhere.

## 16. Database normalization

Normalization organizes data to reduce duplication and prevent inconsistent updates.

Imagine storing this data in one table:

| student_name | department_name  | department_phone |
| ------------ | ---------------- | ---------------- |
| Ahmed        | Computer Science | 123              |
| Mona         | Computer Science | 123              |
| Omar         | Business         | 456              |

The department information is repeated. If the phone changes, multiple rows must be updated.

A better design separates students and departments:

- `departments(id, name, phone)`
- `students(id, name, department_id)`

### Normal forms to know

| Normal form | Main idea                                                    |
| ----------- | ------------------------------------------------------------ |
| 1NF         | Each field contains a single value; no repeating groups      |
| 2NF         | In 1NF, and every non-key attribute depends on the whole candidate key |
| 3NF         | In 2NF, and non-key attributes don't depend transitively on a candidate key |

For most CRUD applications, understanding 1NF, 2NF and 3NF is a good foundation.

Normalization improves consistency, but some systems deliberately denormalize data to optimize read performance.

## 17. Transactions and ACID

Transactions are essential when a business operation modifies multiple records.

Consider transferring 100 EGP between two bank accounts. You must subtract from one account and add to the other. Both changes should succeed together.

```
START TRANSACTION;

UPDATE accounts
SET balance = balance - 100
WHERE id = 1;

UPDATE accounts
SET balance = balance + 100
WHERE id = 2;

COMMIT;
```

If the operation fails before committing, you can roll back the transaction:

```
ROLLBACK;
```

In application code, you generally choose to commit or roll back according to the operation's outcome.

### ACID properties

| Property    | Meaning                                                      |
| ----------- | ------------------------------------------------------------ |
| Atomicity   | All changes succeed together or are rolled back              |
| Consistency | The transaction preserves database constraints and invariants |
| Isolation   | Concurrent transactions are controlled to limit interference |
| Durability  | Committed changes survive failures according to the database's durability guarantees |

### SAVEPOINT

```
START TRANSACTION;

UPDATE accounts
SET balance = balance - 100
WHERE id = 1;

SAVEPOINT after_debit;

UPDATE accounts
SET balance = balance + 100
WHERE id = 2;

ROLLBACK TO SAVEPOINT after_debit;

COMMIT;
```

This example intentionally rolls back the credit while retaining the debit, so it would not be a valid completed transfer. It demonstrates how savepoints work, not how to implement banking logic.

Backend lesson: Real transfers must preserve business invariants. Roll back the entire transaction when the transfer cannot be completed.

## 18. Concurrency and isolation levels

Multiple users can access the same database simultaneously. Concurrency control helps protect data from conflicting operations.

Common isolation levels:

| Level              | General behavior                                             |
| ------------------ | ------------------------------------------------------------ |
| `READ UNCOMMITTED` | May permit dirty reads                                       |
| `READ COMMITTED`   | Prevents dirty reads                                         |
| `REPEATABLE READ`  | Prevents non-repeatable reads; other behavior depends on the database |
| `SERIALIZABLE`     | Strongest standard isolation, with increased contention or retries possible |

In MySQL InnoDB, `REPEATABLE READ` is the default isolation level in typical configurations.

### Example: locking a row

```
START TRANSACTION;

SELECT balance
FROM accounts
WHERE id = 1
FOR UPDATE;

-- Validate the balance and perform the update.

COMMIT;
```

`FOR UPDATE` requests a lock on selected rows, subject to the database engine and transaction rules. This can help prevent two requests from concurrently spending the same funds.

In Spring Boot, transaction management is commonly handled using `@Transactional`, but you still need to understand isolation, locking, and retry behavior.

## 19. Database security and permissions

Create a database user and grant permissions using MySQL syntax:

```
CREATE USER 'app_user'@'localhost'
IDENTIFIED BY 'replace_with_a_strong_secret';

GRANT SELECT, INSERT, UPDATE, DELETE
ON university_db.*
TO 'app_user'@'localhost';
```

Give application users only the permissions they need. Use a secret manager or protected environment configuration for credentials rather than committing passwords to GitHub.

### SQL injection prevention

Unsafe SQL construction:

```
String sql = "SELECT * FROM students WHERE name = '" + name + "'";
```

If the input contains malicious SQL, concatenation can change the query's meaning.

Safer JDBC approach:

```
String sql = "SELECT id, name FROM students WHERE name = ?";

try (PreparedStatement stmt = connection.prepareStatement(sql)) {
    stmt.setString(1, name);

    try (ResultSet rs = stmt.executeQuery()) {
        while (rs.next()) {
            System.out.println(rs.getString("name"));
        }
    }
}
```

A `PreparedStatement` binds the value as data rather than treating it as SQL syntax. Parameterized queries are a fundamental security practice.

## 20. Stored procedures and triggers

These are useful database concepts to recognize, although they aren't the first priority for most Spring Boot beginners.

### Stored procedure

A stored procedure is a named set of SQL statements stored in the database.

```
DELIMITER //

CREATE PROCEDURE GetAllStudents()
BEGIN
    SELECT id, name, email
    FROM students;
END //

DELIMITER ;

CALL GetAllStudents();
```

`DELIMITER` is a MySQL client command used to change how statement endings are interpreted while defining the procedure.

### Trigger

A trigger runs automatically in response to a database event.

```
CREATE TRIGGER before_student_insert
BEFORE INSERT ON students
FOR EACH ROW
SET NEW.name = TRIM(NEW.name);
```

This MySQL trigger removes leading and trailing spaces from a student's name before insertion.

Triggers can be useful for auditing and enforcing certain database behaviors, but hidden side effects can make application debugging harder. Don't use them automatically when ordinary application logic or constraints are sufficient.

## 21. How SQL connects to Java Spring Boot

This is where your SQL knowledge becomes practical backend engineering.

Client / Frontend

Sends HTTP request

Controller

Receives the API request

Service

Business logic and validation

Repository

JPA / Hibernate / SQL

MySQL database

Stores and retrieves data

### Technologies to understand

| Technology          | Purpose                                              |
| ------------------- | ---------------------------------------------------- |
| SQL                 | Query and manipulate relational data                 |
| JDBC                | Java API for working directly with databases         |
| `PreparedStatement` | Executes parameterized SQL                           |
| JPA                 | Java specification for object-relational persistence |
| Hibernate           | Popular JPA implementation                           |
| Spring Data JPA     | Simplifies repository and database operations        |
| Flyway / Liquibase  | Database schema migrations                           |
| HikariCP            | Common JDBC connection pool                          |

### Example Spring Data JPA repository

Suppose you have a Java `Student` entity mapped to the `students` table.

```
import org.springframework.data.jpa.repository.JpaRepository;
import java.util.List;

public interface StudentRepository
        extends JpaRepository<Student, Long> {

    List<Student> findByAgeGreaterThanEqual(int age);
}
```

Spring Data JPA can derive the query from the method name. Conceptually, it retrieves students whose age is greater than or equal to the supplied value.

A corresponding SQL query would resemble:

```
SELECT *
FROM students
WHERE age >= ?;
```

The exact SQL generated by Hibernate depends on the entity mapping, database dialect and framework configuration.

Important: Learn SQL even when using JPA. You need it to debug incorrect queries, understand joins, diagnose performance issues, and design efficient database schemas.

## 22. SQL interview questions you should master

1. What is the difference between WHERE and HAVING?
2. What is the difference between PRIMARY KEY and UNIQUE?
3. What is the difference between INNER JOIN and LEFT JOIN?
4. What is an index?
5. What is normalization?
6. What is a transaction?
7. What is SQL injection and how do you prevent it?
8. What is the difference between DELETE and DROP?
9. What is a foreign key?
10. What is the N+1 query problem in JPA?



## 24. Your learning order

Stage 1 — SQL fundamentals

CREATE, ALTER, INSERT, UPDATE, DELETE, SELECT, WHERE, ORDER BY and LIMIT.

Stage 2 — Querying and data modeling

JOINs, aggregates, subqueries, primary keys, foreign keys, normalization and constraints.

Stage 3 — Intermediate and advanced SQL

Indexes, execution plans, CTEs, transactions, isolation and query optimization.

Stage 4 — Java backend integration

JDBC, Spring Data JPA, Hibernate, entity relationships, transactions, pagination, migrations and API error handling.


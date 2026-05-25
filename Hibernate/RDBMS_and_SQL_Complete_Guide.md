# RDBMS Basics and SQL Queries - Complete Guide

## Table of Contents
1. [Database Fundamentals](#database-fundamentals)
2. [RDBMS Concepts](#rdbms-concepts)
3. [SQL Basics](#sql-basics)
4. [Advanced SQL Queries](#advanced-sql-queries)
5. [Database Design](#database-design)
6. [Performance Optimization](#performance-optimization)
7. [Transactions and ACID Properties](#transactions-and-acid-properties)
8. [Best Practices](#best-practices)

---

## 1. Database Fundamentals

### What is a Database?
A database is an organized collection of structured data stored electronically in a computer system. It allows for efficient storage, retrieval, modification, and deletion of data.

### Types of Databases
- **Relational Databases (RDBMS)**: MySQL, PostgreSQL, Oracle, SQL Server
- **NoSQL Databases**: MongoDB, Cassandra, Redis
- **NewSQL Databases**: CockroachDB, Google Spanner
- **Graph Databases**: Neo4j, Amazon Neptune

### Why Use RDBMS?
- Structured data with relationships
- Data integrity and consistency
- ACID compliance
- Powerful query language (SQL)
- Scalability and security
- Data normalization

---

## 2. RDBMS Concepts

### 2.1 Core Components

#### Tables (Relations)
- Basic storage unit in RDBMS
- Organized in rows and columns
- Each table represents an entity

```sql
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    email VARCHAR(100),
    hire_date DATE,
    salary DECIMAL(10, 2),
    department_id INT
);
```

#### Rows (Tuples/Records)
- Individual entries in a table
- Represents a single instance of an entity

#### Columns (Attributes/Fields)
- Defines the data type and properties
- Each column has a specific data type

### 2.2 Keys

#### Primary Key
- Uniquely identifies each record
- Cannot be NULL
- Must be unique

```sql
CREATE TABLE students (
    student_id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100) UNIQUE
);
```

#### Foreign Key
- Links two tables together
- References primary key in another table
- Maintains referential integrity

```sql
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);
```

#### Candidate Key
- Attributes that can uniquely identify a record
- Can become primary key

#### Composite Key
- Primary key made up of multiple columns

```sql
CREATE TABLE enrollment (
    student_id INT,
    course_id INT,
    enrollment_date DATE,
    PRIMARY KEY (student_id, course_id)
);
```

#### Unique Key
- Ensures all values in a column are unique
- Can be NULL (unlike primary key)

### 2.3 Relationships

#### One-to-One (1:1)
```sql
-- Person and Passport
CREATE TABLE persons (
    person_id INT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE passports (
    passport_id INT PRIMARY KEY,
    person_id INT UNIQUE,
    passport_number VARCHAR(20),
    FOREIGN KEY (person_id) REFERENCES persons(person_id)
);
```

#### One-to-Many (1:N)
```sql
-- Department and Employees
CREATE TABLE departments (
    dept_id INT PRIMARY KEY,
    dept_name VARCHAR(100)
);

CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    emp_name VARCHAR(100),
    dept_id INT,
    FOREIGN KEY (dept_id) REFERENCES departments(dept_id)
);
```

#### Many-to-Many (M:N)
```sql
-- Students and Courses (requires junction table)
CREATE TABLE students (
    student_id INT PRIMARY KEY,
    student_name VARCHAR(100)
);

CREATE TABLE courses (
    course_id INT PRIMARY KEY,
    course_name VARCHAR(100)
);

CREATE TABLE enrollments (
    student_id INT,
    course_id INT,
    enrollment_date DATE,
    grade CHAR(2),
    PRIMARY KEY (student_id, course_id),
    FOREIGN KEY (student_id) REFERENCES students(student_id),
    FOREIGN KEY (course_id) REFERENCES courses(course_id)
);
```

### 2.4 Constraints

```sql
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100) NOT NULL,
    price DECIMAL(10, 2) CHECK (price > 0),
    stock_quantity INT DEFAULT 0,
    category VARCHAR(50),
    supplier_id INT,
    email VARCHAR(100) UNIQUE,
    FOREIGN KEY (supplier_id) REFERENCES suppliers(supplier_id)
);
```

**Types of Constraints:**
- `PRIMARY KEY`: Unique identifier
- `FOREIGN KEY`: Referential integrity
- `UNIQUE`: No duplicate values
- `NOT NULL`: Must have a value
- `CHECK`: Custom validation rule
- `DEFAULT`: Default value if not specified

---

## 3. SQL Basics

### 3.1 Data Definition Language (DDL)

#### CREATE
```sql
-- Create Database
CREATE DATABASE company_db;

-- Create Table
CREATE TABLE employees (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE,
    age INT CHECK (age >= 18),
    salary DECIMAL(10, 2),
    hire_date DATE DEFAULT CURRENT_DATE
);
```

#### ALTER
```sql
-- Add column
ALTER TABLE employees ADD COLUMN phone VARCHAR(15);

-- Modify column
ALTER TABLE employees MODIFY COLUMN salary DECIMAL(12, 2);

-- Drop column
ALTER TABLE employees DROP COLUMN age;

-- Add constraint
ALTER TABLE employees ADD CONSTRAINT chk_salary CHECK (salary > 0);
```

#### DROP
```sql
-- Drop table
DROP TABLE employees;

-- Drop database
DROP DATABASE company_db;

-- Drop constraint
ALTER TABLE employees DROP CONSTRAINT chk_salary;
```

#### TRUNCATE
```sql
-- Remove all records (faster than DELETE)
TRUNCATE TABLE employees;
```

### 3.2 Data Manipulation Language (DML)

#### INSERT
```sql
-- Single row insert
INSERT INTO employees (name, email, salary, hire_date)
VALUES ('John Doe', 'john@example.com', 50000, '2024-01-15');

-- Multiple rows insert
INSERT INTO employees (name, email, salary) VALUES
    ('Jane Smith', 'jane@example.com', 60000),
    ('Bob Johnson', 'bob@example.com', 55000),
    ('Alice Williams', 'alice@example.com', 65000);

-- Insert from another table
INSERT INTO employees_backup
SELECT * FROM employees WHERE salary > 50000;
```

#### SELECT
```sql
-- Basic select
SELECT * FROM employees;

-- Select specific columns
SELECT name, email, salary FROM employees;

-- Select with alias
SELECT name AS employee_name, salary AS annual_salary FROM employees;

-- Select distinct values
SELECT DISTINCT department FROM employees;

-- Select with calculations
SELECT name, salary, salary * 12 AS annual_salary FROM employees;
```

#### UPDATE
```sql
-- Update single record
UPDATE employees 
SET salary = 55000 
WHERE id = 1;

-- Update multiple records
UPDATE employees 
SET salary = salary * 1.1 
WHERE department = 'IT';

-- Update with subquery
UPDATE employees 
SET salary = (SELECT AVG(salary) FROM employees WHERE department = 'HR')
WHERE department = 'HR' AND salary < 40000;
```

#### DELETE
```sql
-- Delete specific records
DELETE FROM employees WHERE id = 1;

-- Delete with condition
DELETE FROM employees WHERE hire_date < '2020-01-01';

-- Delete all records
DELETE FROM employees;
```

### 3.3 Data Query Language (DQL)

#### WHERE Clause
```sql
-- Comparison operators
SELECT * FROM employees WHERE salary > 50000;
SELECT * FROM employees WHERE department = 'IT';
SELECT * FROM employees WHERE age BETWEEN 25 AND 35;

-- Logical operators
SELECT * FROM employees WHERE salary > 50000 AND department = 'IT';
SELECT * FROM employees WHERE department = 'IT' OR department = 'HR';
SELECT * FROM employees WHERE NOT department = 'IT';

-- Pattern matching
SELECT * FROM employees WHERE name LIKE 'J%';  -- Starts with J
SELECT * FROM employees WHERE name LIKE '%son'; -- Ends with son
SELECT * FROM employees WHERE name LIKE '%a%';  -- Contains a

-- IN operator
SELECT * FROM employees WHERE department IN ('IT', 'HR', 'Finance');

-- NULL checks
SELECT * FROM employees WHERE manager_id IS NULL;
SELECT * FROM employees WHERE email IS NOT NULL;
```

#### ORDER BY
```sql
-- Ascending order (default)
SELECT * FROM employees ORDER BY salary;

-- Descending order
SELECT * FROM employees ORDER BY salary DESC;

-- Multiple columns
SELECT * FROM employees ORDER BY department ASC, salary DESC;
```

#### LIMIT and OFFSET
```sql
-- Get first 10 records
SELECT * FROM employees LIMIT 10;

-- Pagination (skip 10, get next 10)
SELECT * FROM employees LIMIT 10 OFFSET 10;

-- Alternative syntax (MySQL)
SELECT * FROM employees LIMIT 10, 10;
```

#### GROUP BY
```sql
-- Basic grouping
SELECT department, COUNT(*) AS employee_count
FROM employees
GROUP BY department;

-- Multiple columns
SELECT department, job_title, AVG(salary) AS avg_salary
FROM employees
GROUP BY department, job_title;

-- With aggregate functions
SELECT department, 
       COUNT(*) AS total_employees,
       AVG(salary) AS avg_salary,
       MAX(salary) AS max_salary,
       MIN(salary) AS min_salary
FROM employees
GROUP BY department;
```

#### HAVING
```sql
-- Filter grouped results
SELECT department, AVG(salary) AS avg_salary
FROM employees
GROUP BY department
HAVING AVG(salary) > 50000;

-- Multiple conditions
SELECT department, COUNT(*) AS emp_count
FROM employees
GROUP BY department
HAVING COUNT(*) > 5 AND AVG(salary) > 45000;
```

---

## 4. Advanced SQL Queries

### 4.1 Joins

#### INNER JOIN
```sql
-- Returns only matching records from both tables
SELECT e.name, e.salary, d.department_name
FROM employees e
INNER JOIN departments d ON e.department_id = d.dept_id;
```

#### LEFT JOIN (LEFT OUTER JOIN)
```sql
-- Returns all records from left table and matching from right
SELECT e.name, d.department_name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.dept_id;
```

#### RIGHT JOIN (RIGHT OUTER JOIN)
```sql
-- Returns all records from right table and matching from left
SELECT e.name, d.department_name
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.dept_id;
```

#### FULL OUTER JOIN
```sql
-- Returns all records when there's a match in either table
SELECT e.name, d.department_name
FROM employees e
FULL OUTER JOIN departments d ON e.department_id = d.dept_id;

-- MySQL alternative (no native FULL OUTER JOIN)
SELECT e.name, d.department_name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.dept_id
UNION
SELECT e.name, d.department_name
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.dept_id;
```

#### CROSS JOIN
```sql
-- Cartesian product of both tables
SELECT e.name, d.department_name
FROM employees e
CROSS JOIN departments d;
```

#### SELF JOIN
```sql
-- Join table to itself
SELECT e1.name AS employee, e2.name AS manager
FROM employees e1
LEFT JOIN employees e2 ON e1.manager_id = e2.employee_id;
```

### 4.2 Subqueries

#### Single Row Subquery
```sql
-- Get employees with salary greater than average
SELECT name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);
```

#### Multiple Row Subquery
```sql
-- Using IN
SELECT name, department_id
FROM employees
WHERE department_id IN (SELECT dept_id FROM departments WHERE location = 'New York');

-- Using ANY
SELECT name, salary
FROM employees
WHERE salary > ANY (SELECT salary FROM employees WHERE department = 'IT');

-- Using ALL
SELECT name, salary
FROM employees
WHERE salary > ALL (SELECT salary FROM employees WHERE department = 'HR');
```

#### Correlated Subquery
```sql
-- Subquery references outer query
SELECT e1.name, e1.salary, e1.department_id
FROM employees e1
WHERE salary > (
    SELECT AVG(salary) 
    FROM employees e2 
    WHERE e2.department_id = e1.department_id
);
```

#### Subquery in FROM Clause
```sql
SELECT dept, avg_salary
FROM (
    SELECT department_id AS dept, AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department_id
) AS dept_avg
WHERE avg_salary > 50000;
```

### 4.3 Set Operations

#### UNION
```sql
-- Combines results and removes duplicates
SELECT name FROM employees WHERE department = 'IT'
UNION
SELECT name FROM employees WHERE salary > 60000;
```

#### UNION ALL
```sql
-- Combines results keeping duplicates
SELECT name FROM employees WHERE department = 'IT'
UNION ALL
SELECT name FROM employees WHERE salary > 60000;
```

#### INTERSECT
```sql
-- Returns common records
SELECT name FROM employees WHERE department = 'IT'
INTERSECT
SELECT name FROM employees WHERE salary > 60000;
```

#### EXCEPT (MINUS)
```sql
-- Returns records from first query not in second
SELECT name FROM employees WHERE department = 'IT'
EXCEPT
SELECT name FROM employees WHERE salary > 60000;
```

### 4.4 Aggregate Functions

```sql
-- COUNT
SELECT COUNT(*) AS total_employees FROM employees;
SELECT COUNT(DISTINCT department_id) AS unique_departments FROM employees;

-- SUM
SELECT SUM(salary) AS total_payroll FROM employees;

-- AVG
SELECT AVG(salary) AS average_salary FROM employees;

-- MIN and MAX
SELECT MIN(salary) AS lowest_salary, MAX(salary) AS highest_salary FROM employees;

-- String aggregation
SELECT department_id, GROUP_CONCAT(name) AS employee_list
FROM employees
GROUP BY department_id;
```

### 4.5 Window Functions

```sql
-- ROW_NUMBER
SELECT name, salary, department,
       ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) AS rank
FROM employees;

-- RANK (with gaps)
SELECT name, salary,
       RANK() OVER (ORDER BY salary DESC) AS rank
FROM employees;

-- DENSE_RANK (without gaps)
SELECT name, salary,
       DENSE_RANK() OVER (ORDER BY salary DESC) AS dense_rank
FROM employees;

-- NTILE (divide into buckets)
SELECT name, salary,
       NTILE(4) OVER (ORDER BY salary) AS quartile
FROM employees;

-- LAG and LEAD
SELECT name, salary,
       LAG(salary) OVER (ORDER BY hire_date) AS previous_salary,
       LEAD(salary) OVER (ORDER BY hire_date) AS next_salary
FROM employees;

-- Running total
SELECT name, salary,
       SUM(salary) OVER (ORDER BY hire_date) AS running_total
FROM employees;

-- Moving average
SELECT name, salary,
       AVG(salary) OVER (ORDER BY hire_date ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS moving_avg
FROM employees;
```

### 4.6 Common Table Expressions (CTE)

```sql
-- Basic CTE
WITH high_earners AS (
    SELECT name, salary, department
    FROM employees
    WHERE salary > 70000
)
SELECT * FROM high_earners WHERE department = 'IT';

-- Multiple CTEs
WITH 
dept_avg AS (
    SELECT department_id, AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department_id
),
dept_names AS (
    SELECT dept_id, dept_name
    FROM departments
)
SELECT d.dept_name, da.avg_salary
FROM dept_avg da
JOIN dept_names d ON da.department_id = d.dept_id;

-- Recursive CTE (organizational hierarchy)
WITH RECURSIVE employee_hierarchy AS (
    -- Anchor member
    SELECT employee_id, name, manager_id, 1 AS level
    FROM employees
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- Recursive member
    SELECT e.employee_id, e.name, e.manager_id, eh.level + 1
    FROM employees e
    JOIN employee_hierarchy eh ON e.manager_id = eh.employee_id
)
SELECT * FROM employee_hierarchy;
```

### 4.7 CASE Statements

```sql
-- Simple CASE
SELECT name, salary,
    CASE 
        WHEN salary < 40000 THEN 'Low'
        WHEN salary BETWEEN 40000 AND 70000 THEN 'Medium'
        WHEN salary > 70000 THEN 'High'
        ELSE 'Unknown'
    END AS salary_category
FROM employees;

-- Searched CASE
SELECT name, department,
    CASE department
        WHEN 'IT' THEN 'Technology'
        WHEN 'HR' THEN 'Human Resources'
        WHEN 'FIN' THEN 'Finance'
        ELSE 'Other'
    END AS department_full_name
FROM employees;

-- CASE in aggregate
SELECT department,
    COUNT(CASE WHEN salary > 60000 THEN 1 END) AS high_earners,
    COUNT(CASE WHEN salary <= 60000 THEN 1 END) AS low_earners
FROM employees
GROUP BY department;
```

### 4.8 Pivot and Unpivot

```sql
-- Pivot (MySQL example)
SELECT 
    employee_id,
    MAX(CASE WHEN skill = 'Java' THEN proficiency END) AS Java,
    MAX(CASE WHEN skill = 'Python' THEN proficiency END) AS Python,
    MAX(CASE WHEN skill = 'SQL' THEN proficiency END) AS SQL
FROM employee_skills
GROUP BY employee_id;

-- Unpivot (MySQL example)
SELECT employee_id, 'Java' AS skill, Java AS proficiency FROM skills_pivot
UNION ALL
SELECT employee_id, 'Python' AS skill, Python AS proficiency FROM skills_pivot
UNION ALL
SELECT employee_id, 'SQL' AS skill, SQL AS proficiency FROM skills_pivot;
```

---

## 5. Database Design

### 5.1 Normalization

#### First Normal Form (1NF)
- Eliminate repeating groups
- Create separate table for each set of related data
- Identify each record with a primary key

**Before 1NF:**
```
Employee_ID | Name | Phone_Numbers
1 | John | 555-1234, 555-5678
```

**After 1NF:**
```
Employee_ID | Name | Phone_Number
1 | John | 555-1234
1 | John | 555-5678
```

#### Second Normal Form (2NF)
- Must be in 1NF
- Remove partial dependencies
- Non-key attributes must depend on entire primary key

**Before 2NF:**
```
Order_ID | Product_ID | Product_Name | Quantity
1 | 101 | Widget | 5
```

**After 2NF:**
```
Orders: Order_ID | Product_ID | Quantity
Products: Product_ID | Product_Name
```

#### Third Normal Form (3NF)
- Must be in 2NF
- Remove transitive dependencies
- Non-key attributes must depend only on primary key

**Before 3NF:**
```
Employee_ID | Name | Department_ID | Department_Name
1 | John | 10 | IT
```

**After 3NF:**
```
Employees: Employee_ID | Name | Department_ID
Departments: Department_ID | Department_Name
```

#### Boyce-Codd Normal Form (BCNF)
- Stricter version of 3NF
- Every determinant must be a candidate key

### 5.2 Denormalization
When to denormalize:
- Read-heavy applications
- Performance optimization
- Reduce complex joins
- Data warehousing

```sql
-- Denormalized for reporting
CREATE TABLE sales_report (
    sale_id INT,
    customer_name VARCHAR(100),
    product_name VARCHAR(100),
    category VARCHAR(50),
    sale_amount DECIMAL(10, 2),
    sale_date DATE
);
```

---

## 6. Performance Optimization

### 6.1 Indexes

```sql
-- Create index
CREATE INDEX idx_employee_name ON employees(name);

-- Composite index
CREATE INDEX idx_dept_salary ON employees(department_id, salary);

-- Unique index
CREATE UNIQUE INDEX idx_email ON employees(email);

-- Full-text index
CREATE FULLTEXT INDEX idx_description ON products(description);

-- Drop index
DROP INDEX idx_employee_name ON employees;

-- Show indexes
SHOW INDEX FROM employees;
```

**When to use indexes:**
- Columns used in WHERE clauses
- Columns used in JOIN conditions
- Columns used in ORDER BY
- Foreign key columns

**When NOT to use indexes:**
- Small tables
- Columns with low cardinality
- Frequently updated columns
- Wide columns

### 6.2 Query Optimization

```sql
-- Use EXPLAIN to analyze queries
EXPLAIN SELECT * FROM employees WHERE department_id = 10;

-- Avoid SELECT *
SELECT employee_id, name, salary FROM employees;

-- Use EXISTS instead of IN for large datasets
SELECT * FROM employees e
WHERE EXISTS (SELECT 1 FROM departments d WHERE d.dept_id = e.department_id);

-- Avoid functions in WHERE clause
-- Bad: WHERE YEAR(hire_date) = 2024
-- Good: WHERE hire_date >= '2024-01-01' AND hire_date < '2025-01-01'

-- Use LIMIT for large result sets
SELECT * FROM employees ORDER BY salary DESC LIMIT 10;
```

### 6.3 Views

```sql
-- Create view
CREATE VIEW high_earners AS
SELECT name, salary, department
FROM employees
WHERE salary > 70000;

-- Use view
SELECT * FROM high_earners;

-- Materialized view (PostgreSQL)
CREATE MATERIALIZED VIEW dept_summary AS
SELECT department_id, COUNT(*) AS emp_count, AVG(salary) AS avg_salary
FROM employees
GROUP BY department_id;

-- Refresh materialized view
REFRESH MATERIALIZED VIEW dept_summary;

-- Drop view
DROP VIEW high_earners;
```

### 6.4 Stored Procedures

```sql
-- Create stored procedure
DELIMITER //
CREATE PROCEDURE GetEmployeesByDept(IN dept_id INT)
BEGIN
    SELECT * FROM employees WHERE department_id = dept_id;
END //
DELIMITER ;

-- Call procedure
CALL GetEmployeesByDept(10);

-- Procedure with OUT parameter
DELIMITER //
CREATE PROCEDURE GetEmployeeCount(IN dept_id INT, OUT emp_count INT)
BEGIN
    SELECT COUNT(*) INTO emp_count FROM employees WHERE department_id = dept_id;
END //
DELIMITER ;

-- Call with OUT parameter
CALL GetEmployeeCount(10, @count);
SELECT @count;
```

### 6.5 Triggers

```sql
-- Create trigger
DELIMITER //
CREATE TRIGGER before_employee_update
BEFORE UPDATE ON employees
FOR EACH ROW
BEGIN
    IF NEW.salary < OLD.salary THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Salary cannot be decreased';
    END IF;
END //
DELIMITER ;

-- Audit trigger
CREATE TRIGGER after_employee_insert
AFTER INSERT ON employees
FOR EACH ROW
INSERT INTO employee_audit (employee_id, action, action_date)
VALUES (NEW.employee_id, 'INSERT', NOW());

-- Drop trigger
DROP TRIGGER before_employee_update;
```

---

## 7. Transactions and ACID Properties

### 7.1 ACID Properties

**Atomicity**: All operations in a transaction succeed or fail together
**Consistency**: Data remains valid after transaction
**Isolation**: Concurrent transactions don't interfere
**Durability**: Completed transactions persist even after system failure

### 7.2 Transaction Control

```sql
-- Begin transaction
START TRANSACTION;
-- or
BEGIN;

-- Execute statements
UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
UPDATE accounts SET balance = balance + 100 WHERE account_id = 2;

-- Commit transaction
COMMIT;

-- Rollback transaction
ROLLBACK;

-- Savepoint
START TRANSACTION;
UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
SAVEPOINT sp1;
UPDATE accounts SET balance = balance + 100 WHERE account_id = 2;
ROLLBACK TO sp1;
COMMIT;
```

### 7.3 Isolation Levels

```sql
-- Set isolation level
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- Example
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
START TRANSACTION;
SELECT * FROM accounts WHERE account_id = 1 FOR UPDATE;
UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
COMMIT;
```

### 7.4 Locking

```sql
-- Shared lock (read)
SELECT * FROM employees WHERE department_id = 10 LOCK IN SHARE MODE;

-- Exclusive lock (write)
SELECT * FROM employees WHERE employee_id = 1 FOR UPDATE;

-- Row-level locking
START TRANSACTION;
SELECT * FROM accounts WHERE account_id = 1 FOR UPDATE;
UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
COMMIT;
```

---

## 8. Best Practices

### 8.1 Naming Conventions

```sql
-- Tables: plural, lowercase with underscores
employees, customer_orders, product_categories

-- Columns: singular, lowercase with underscores
employee_id, first_name, created_at

-- Primary keys: table_name_id
employee_id, order_id, product_id

-- Foreign keys: referenced_table_name_id
department_id, customer_id, category_id

-- Indexes: idx_table_column
idx_employees_department, idx_orders_customer

-- Constraints: type_table_column
pk_employees_id, fk_orders_customer, chk_salary_positive
```

### 8.2 Security Best Practices

```sql
-- Use parameterized queries (prevent SQL injection)
-- Bad: "SELECT * FROM users WHERE username = '" + input + "'"
-- Good: Use prepared statements

-- Grant minimum privileges
GRANT SELECT ON database.table TO 'user'@'localhost';

-- Encrypt sensitive data
CREATE TABLE users (
    user_id INT PRIMARY KEY,
    username VARCHAR(50),
    password_hash VARCHAR(255) -- Never store plain passwords
);

-- Use SSL/TLS for connections
-- mysql --ssl-mode=REQUIRED -u user -p
```

### 8.3 Data Integrity

```sql
-- Use constraints
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT NOT NULL,
    order_date DATE NOT NULL DEFAULT CURRENT_DATE,
    total_amount DECIMAL(10, 2) CHECK (total_amount >= 0),
    status VARCHAR(20) CHECK (status IN ('pending', 'completed', 'cancelled')),
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);

-- Use triggers for complex validation
-- Use transactions for related operations
-- Regular backups
```

### 8.4 Performance Tips

1. **Index strategically**: Not too many, not too few
2. **Avoid SELECT ***: Select only needed columns
3. **Use JOINs efficiently**: Proper join order matters
4. **Limit result sets**: Use LIMIT and pagination
5. **Optimize subqueries**: Use JOINs when possible
6. **Use EXPLAIN**: Analyze query execution plans
7. **Regular maintenance**: Update statistics, rebuild indexes
8. **Connection pooling**: Reuse database connections
9. **Caching**: Cache frequently accessed data
10. **Partitioning**: For very large tables

### 8.5 Common Patterns

```sql
-- Pagination
SELECT * FROM products
ORDER BY product_id
LIMIT 20 OFFSET 40; -- Page 3 (20 per page)

-- Soft delete
ALTER TABLE employees ADD COLUMN deleted_at DATETIME NULL;
UPDATE employees SET deleted_at = NOW() WHERE employee_id = 1;
SELECT * FROM employees WHERE deleted_at IS NULL;

-- Audit trail
CREATE TABLE audit_log (
    audit_id INT AUTO_INCREMENT PRIMARY KEY,
    table_name VARCHAR(50),
    record_id INT,
    action VARCHAR(10),
    user_id INT,
    timestamp DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Timestamps
ALTER TABLE employees 
ADD COLUMN created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
ADD COLUMN updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP;

-- Upsert (Insert or Update)
INSERT INTO products (product_id, name, price)
VALUES (1, 'Widget', 9.99)
ON DUPLICATE KEY UPDATE price = 9.99;
```

---

## Additional SQL Features

### Date and Time Functions

```sql
-- Current date/time
SELECT CURRENT_DATE, CURRENT_TIME, CURRENT_TIMESTAMP, NOW();

-- Date arithmetic
SELECT DATE_ADD(CURRENT_DATE, INTERVAL 7 DAY);
SELECT DATE_SUB(CURRENT_DATE, INTERVAL 1 MONTH);
SELECT DATEDIFF('2024-12-31', '2024-01-01');

-- Extract parts
SELECT YEAR(hire_date), MONTH(hire_date), DAY(hire_date) FROM employees;

-- Format dates
SELECT DATE_FORMAT(hire_date, '%Y-%m-%d') FROM employees;
```

### String Functions

```sql
-- Concatenation
SELECT CONCAT(first_name, ' ', last_name) AS full_name FROM employees;

-- Substring
SELECT SUBSTRING(name, 1, 5) FROM employees;

-- Case conversion
SELECT UPPER(name), LOWER(email) FROM employees;

-- Trim
SELECT TRIM(name), LTRIM(name), RTRIM(name) FROM employees;

-- Length
SELECT LENGTH(name), CHAR_LENGTH(name) FROM employees;

-- Replace
SELECT REPLACE(email, '@oldomain.com', '@newdomain.com') FROM employees;
```

### Math Functions

```sql
-- Basic operations
SELECT ROUND(salary, 2), CEIL(salary), FLOOR(salary) FROM employees;

-- Aggregates
SELECT ABS(difference), POWER(value, 2), SQRT(value) FROM calculations;

-- Random
SELECT RAND(), FLOOR(RAND() * 100);
```

### NULL Handling

```sql
-- COALESCE (returns first non-null)
SELECT COALESCE(phone, email, 'No contact') FROM employees;

-- IFNULL / ISNULL
SELECT IFNULL(commission, 0) FROM employees;

-- NULLIF (returns NULL if equal)
SELECT NULLIF(value, 0) FROM data; -- Prevents division by zero
```

---

## Summary

This guide covers:
1. Database and RDBMS fundamentals
2. Core concepts: tables, keys, relationships, constraints
3. SQL basics: DDL, DML, DQL commands
4. Advanced queries: joins, subqueries, window functions, CTEs
5. Database design: normalization and denormalization
6. Performance: indexes, optimization, views, procedures
7. Transactions: ACID properties, isolation levels, locking
8. Best practices: naming, security, integrity, performance

**Key Takeaways:**
- Start with proper database design and normalization
- Use constraints to maintain data integrity
- Write clear, readable SQL with meaningful names
- Optimize queries and use indexes wisely
- Always use transactions for related operations
- Follow security best practices
- Regular maintenance and monitoring
- Test queries with EXPLAIN before production

**Next Steps:**
- Practice with real-world datasets
- Learn database-specific features (MySQL, PostgreSQL, etc.)
- Study query execution plans
- Understand backup and recovery strategies
- Explore advanced topics: replication, sharding, partitioning

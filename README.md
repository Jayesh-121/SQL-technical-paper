# SQL Concepts & Database Theory 

# Table of Contents

1. [ACID Properties](#acid-properties)
2. [CAP Theorem](#cap-theorem)
3. [SQL Joins](#sql-joins)
4. [Aggregations & Filters](#aggregations--filters)
5. [Normalization](#normalization)
6. [Indexes](#indexes)
7. [Transactions](#transactions)
8. [Locking Mechanisms](#locking-mechanisms)
9. [Database Isolation Levels](#database-isolation-levels)
10. [Triggers](#triggers)
11. [References](#references)

---

# ACID Properties

## Description

ACID properties define how relational database systems ensure reliable and predictable data manipulation, especially during concurrent operations or system failures. They are the foundation of strong consistency in transactional systems such as banking, inventory management, or ticket reservations.

### Atomicity

Atomicity ensures that a transaction is treated as a single, indivisible unit.
If any statement inside the transaction fails, **all** changes are undone.
This prevents partial updates that could corrupt data integrity (e.g., deducting money from one account but failing to add it to another).

### Consistency

A transaction must always transition the database from one valid state to another, enforcing:

* all constraints (PRIMARY KEY, UNIQUE, CHECK)
* referential integrity
* data type rules
* triggers and business rules  
If any rule is violated during the transaction, it is rolled back automatically.

### Isolation

Isolation determines how and when the changes from one transaction become visible to other concurrent transactions.
It prevents race conditions such as:

* dirty reads
* lost updates
* non-repeatable reads
* phantom rows

Isolation ensures that each transaction executes as if it were the only transaction in the system.

### Durability

Durability guarantees that once a transaction commits, its changes are permanently saved, even if:

* the database crashes
* a server shuts down
* power fails

Databases achieve durability through:

* write-ahead logs (WAL)
* transaction logs
* checkpoints
* disk flushing mechanisms

## Example

```sql
BEGIN;

UPDATE accounts SET balance = balance - 500 WHERE id = 1;
UPDATE accounts SET balance = balance + 500 WHERE id = 2;

COMMIT;
```

If any step fails, the database reverts to its previous state.

---

# CAP Theorem

## Description

The CAP theorem applies to distributed databases and describes the unavoidable trade-offs when a system spans multiple networked nodes. Because network failures (partitions) **will** happen in distributed environments, the system must choose between prioritizing **Consistency** or **Availability**.

### Consistency (C)

All nodes return the same, most up-to-date data.
A read always reflects the most recent successful write.
This may require:

* synchronizing across nodes
* rejecting requests during network issues

### Availability (A)

Every request receives a response (even if stale), even when some nodes are unavailable.
The system prioritizes uptime and responsiveness over strict correctness.

### Partition Tolerance (P)

The system continues functioning even if communication between nodes is disrupted.
Partition tolerance is non-negotiable in real distributed systems.

## Implications

A system must pick **AP** or **CP** during network partitions:

| System | Prioritizes                             | Examples                         |
| ------ | --------------------------------------- | -------------------------------- |
| CP     | Correctness over uptime                 | MongoDB (majority writes), HBase |
| AP     | High availability over strict freshness | Cassandra, DynamoDB              |

---

# SQL Joins

## Description

Joins enable combining related data spread across normalized tables. They rely on relational algebra principles and are fundamental to SQL query design, data analytics, reporting, and database normalization.

### INNER JOIN

Returns rows that have corresponding matches in both tables.

```sql
SELECT e.name, d.department_name
FROM employees e
INNER JOIN departments d
ON e.dept_id = d.id;
```

### LEFT JOIN

Returns all rows from the left table and only the matching rows from the right.

```sql
SELECT e.name, d.department_name
FROM employees e
LEFT JOIN departments d
ON e.dept_id = d.id;
```

### RIGHT JOIN

Returns all rows from the right table and matching rows from the left.
Less commonly used due to readability issues.

### FULL OUTER JOIN

Returns all rows from both tables, aligning matches and leaving NULLs where no match exists.

```sql
SELECT *
FROM a
FULL OUTER JOIN b
ON a.id = b.id;
```

### CROSS JOIN

Produces a Cartesian product (m × n rows).
Useful for generating combinations.

---

# Aggregations & Filters

## Description

SQL aggregations compute summary statistics or group-based results.
Filtering determines which rows or aggregated groups are included.
Together, they allow analysts to derive insights, measure patterns, and produce reports.

### Aggregations

Common functions:

* COUNT()
* SUM()
* AVG()
* MIN() / MAX()

Example:

```sql
SELECT department, COUNT(*) AS total_employees
FROM employees
GROUP BY department;
```

### WHERE Filtering

Row-level filtering (applies before aggregation).

```sql
SELECT *
FROM orders
WHERE amount > 1000;
```

### HAVING Filtering

Used for filtering aggregated groups.

```sql
SELECT department, AVG(salary) AS avg_sal
FROM employees
GROUP BY department
HAVING AVG(salary) > 50000;
```

### Common Pitfalls

* WHERE cannot filter aggregated values
* HAVING is slower (applied after grouping)

---

# Normalization

## Description

Normalization organizes database tables to minimize redundancy, improve update efficiency, and prevent anomalies such as:

* update anomalies
* delete anomalies
* insert anomalies

It ensures that each table has a single, well-defined purpose.

### 1NF

* All values are atomic
* No repeating columns or lists

### 2NF

* Must be in 1NF
* No partial dependency on composite keys

### 3NF

* Must be in 2NF
* No transitive dependencies

### BCNF

A stricter version of 3NF: every determinant must be a candidate key.

Normalization Example:

```
customers(id, name, address)
orders(id, customer_id)
order_items(order_id, product_id)
```

---

# Indexes

## Description

Indexes act like optimized lookup structures that let the database quickly locate rows without scanning entire tables. They are essential for performance, especially on large datasets.

### Advantages

* Faster reads
* Efficient sorting and grouping
* Improved join performance

### Disadvantages

* Slower writes due to index maintenance
* Extra storage required
* Over-indexing harms performance

### Types

* B-tree
* Hash
* Bitmap
* Full-text

Example:

```sql
CREATE INDEX idx_employee_name
ON employees(name);
```

---

# Transactions

## Description

A transaction groups multiple SQL operations into an atomic unit of work. They ensure consistency even when some operations fail.

### Example

```sql
BEGIN;

UPDATE inventory SET stock = stock - 1 WHERE id = 10;
INSERT INTO sales(product_id, date) VALUES (10, NOW());

COMMIT;
```

Rollback:

```sql
ROLLBACK;
```

---

# Locking Mechanisms

## Description

Locks control concurrent data access to maintain integrity.

### Lock Types

* Shared — allows reads but blocks writes
* Exclusive — allows writes but blocks everything else
* Row locks — granular, reduce contention
* Table locks — coarse-grained, used for bulk updates

Example:

```sql
SELECT * FROM accounts WHERE id = 5 FOR UPDATE;
```

---

# Database Isolation Levels

## Description

Isolation levels determine how transactions interact in concurrent environments.

### Levels

* Read Uncommitted
* Read Committed
* Repeatable Read
* Serializable

Example:

```sql
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```

---

# Triggers

## Description

Triggers execute automatically when certain database events occur.

Example:

```sql
CREATE TRIGGER update_timestamp
BEFORE UPDATE ON employees
FOR EACH ROW
EXECUTE PROCEDURE set_updated_at();
```

---

# References

* PostgreSQL Documentation: https://www.postgresql.org/docs/
* MySQL Developer Docs: https://dev.mysql.com/doc/
* SQL Server Docs: https://learn.microsoft.com/en-us/sql/
* SQLite Documentation: https://www.sqlite.org/docs.html

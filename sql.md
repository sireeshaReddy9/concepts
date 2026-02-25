# Database Cheatsheet

---

## ACID

These are the 4 properties every transaction must follow. If any one of them breaks, the DB is unreliable.

| Property | What it means |
|---|---|
| **Atomicity** | All or nothing. Either every operation in a transaction commits, or none of them do. No half-states. |
| **Consistency** | The DB always moves from one valid state to another. Constraints, rules, and cascades must hold. |
| **Isolation** | Transactions were independent of each other.T2 Transaction can start after T1 commits|
| **Durability** | Once committed, it's committed. Even if the server crashes right after, the data stays. |

> My rule: If the power goes out mid-transaction, ACID ensures the DB doesn't end up in a corrupt state.

---

## CAP Theorem

In a distributed system, you can only guarantee **2 out of 3**:

| Property | Meaning |
|---|---|
| **Consistency** | Every read gets the most recent write (or an error) |
| **Availability** | Every request gets a response (not necessarily the latest data) |
| **Partition Tolerance** | System keeps working even if nodes can't talk to each other |

**The hard truth:** Network partitions *will* happen. So the real choice is between **CP** or **AP**.

- **CP** → MySQL, HBase, ZooKeeper (picks consistency, may reject requests during partition)
- **AP** → Cassandra, CouchDB, DynamoDB (picks availability, may return stale data)
- **CA** → Only possible on a single-node system (no partition to tolerate)

---

## Joins

I use joins when I need to combine rows from two or more tables.

```sql
-- INNER JOIN: only rows that match in both tables
SELECT u.name, o.total
FROM users u
INNER JOIN orders o ON u.id = o.user_id;

-- LEFT JOIN: all rows from left table, NULLs if no match on right
SELECT u.name, o.total
FROM users u
LEFT JOIN orders o ON u.id = o.user_id;

-- RIGHT JOIN: all rows from right table, NULLs if no match on left
SELECT u.name, o.total
FROM users u
RIGHT JOIN orders o ON u.id = o.user_id;

-- FULL OUTER JOIN: all rows from both, NULLs where no match
SELECT u.name, o.total
FROM users u
FULL OUTER JOIN orders o ON u.id = o.user_id;

-- SELF JOIN: joining a table with itself (e.g., employee-manager)
SELECT e.name AS employee, m.name AS manager
FROM employees e
JOIN employees m ON e.manager_id = m.id;

-- CROSS JOIN: cartesian product (every row x every row)
SELECT a.color, b.size FROM colors a CROSS JOIN sizes b;
```

> Quick mental model: INNER = intersection, LEFT = all left + matches, FULL OUTER = union.

---

## Aggregations & Filters

### Aggregation Functions

```sql
SELECT
  COUNT(*)           AS total_rows,
  COUNT(salary)      AS non_null_salaries,
  SUM(salary)        AS total_payroll,
  AVG(salary)        AS avg_salary,
  MIN(salary)        AS lowest,
  MAX(salary)        AS highest
FROM employees;
```

### GROUP BY + HAVING

```sql
-- GROUP BY groups rows, HAVING filters groups (WHERE filters rows)
SELECT department, AVG(salary) AS avg_sal
FROM employees
WHERE status = 'active'          -- filters rows BEFORE grouping
GROUP BY department
HAVING AVG(salary) > 70000       -- filters groups AFTER grouping
ORDER BY avg_sal DESC;
```

> Key distinction: `WHERE` runs before aggregation, `HAVING` runs after.

### Window Functions (aggregation without collapsing rows)

```sql
SELECT
  name,
  department,
  salary,
  AVG(salary) OVER (PARTITION BY department) AS dept_avg,
  RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS rank
FROM employees;
```

---

## Normalization

Normalization removes redundancy and prevents anomalies (insert/update/delete).

| Normal Form | Rule |
|---|---|
| **1NF** | Each column holds atomic values. No repeating groups. Every row is unique. |
| **2NF** | Must be in 1NF + every non-key column depends on the *whole* primary key (matters in composite PKs). |
| **3NF** | Must be in 2NF + no transitive dependencies (non-key column shouldn't depend on another non-key column). |
| **BCNF** | Stricter version of 3NF — every determinant must be a candidate key. |

**Example of transitive dependency (breaks 3NF):**
```
Orders(order_id, customer_id, customer_city)
```
`customer_city` depends on `customer_id`, not `order_id` → split into `Customers(customer_id, customer_city)`.

> I normalize to 3NF in most production systems. BCNF only if I need to eliminate all anomalies at the cost of more joins.

---

## Indexes

Indexes speed up reads but slow down writes (because the index also needs updating).

```sql
-- Single column index
CREATE INDEX idx_email ON users(email);

-- Composite index (order matters — leftmost prefix rule)
CREATE INDEX idx_dept_sal ON employees(department, salary);
--  Works for: WHERE department = 'Eng'
--  Works for: WHERE department = 'Eng' AND salary > 50000
--  Doesn't work for: WHERE salary > 50000 (skips first column)

-- Unique index
CREATE UNIQUE INDEX idx_unique_email ON users(email);

-- Partial index (index only a subset)
CREATE INDEX idx_active_users ON users(email) WHERE status = 'active';

-- Drop index
DROP INDEX idx_email;
```

### Types I care about

| Type | Use case |
|---|---|
| **B-Tree** | Default. Good for `=`, `<`, `>`, `BETWEEN`, `ORDER BY` |
| **Hash** | Only equality checks (`=`). Faster for exact lookups |
| **Full-text** | Searching text content (`MATCH`, `AGAINST`) |
| **Composite** | Multiple columns. Follow leftmost prefix rule |
| **Covering index** | Index includes all columns a query needs → no table lookup |

> Rule of thumb: Index columns used in `WHERE`, `JOIN ON`, and `ORDER BY`. Avoid over-indexing on write-heavy tables.

---

## Transactions

A transaction is a unit of work. Either the whole thing succeeds, or it rolls back.

```sql
BEGIN;  -- or START TRANSACTION

UPDATE accounts SET balance = balance - 500 WHERE id = 1;
UPDATE accounts SET balance = balance + 500 WHERE id = 2;

COMMIT;   -- saves everything

-- or if something went wrong:
ROLLBACK; -- undoes everything since BEGIN
```

### Savepoints (partial rollback)

```sql
BEGIN;

INSERT INTO orders (...) VALUES (...);
SAVEPOINT after_order;

INSERT INTO payments (...) VALUES (...);
-- something failed
ROLLBACK TO SAVEPOINT after_order;  -- only undoes the payment insert

COMMIT;
```

---

## Locking Mechanisms

Locks prevent concurrent transactions from corrupting data.

| Lock Type | Description |
|---|---|
| **Shared Lock (S)** | Multiple transactions can read. No writes allowed while held. |
| **Exclusive Lock (X)** | Only one transaction can read/write. Blocks everything else. |
| **Intent Lock** | Signals intent to lock at a finer granularity (used in hierarchical locking). |
| **Row-level lock** | Locks individual rows. Best concurrency. |
| **Table-level lock** | Locks entire table. Simple, but kills concurrency. |
| **Page-level lock** | Locks a page of rows. Between row and table. |
| **Deadlock** | Two transactions waiting on each other's locks forever. DB detects and kills one. |

```sql
-- Explicit row lock in PostgreSQL
SELECT * FROM accounts WHERE id = 1 FOR UPDATE;       -- exclusive lock
SELECT * FROM accounts WHERE id = 1 FOR SHARE;        -- shared lock
```

> I prefer row-level locking in OLTP systems. Table locks are fine for batch jobs where I don't care about concurrency.

**Optimistic vs Pessimistic Locking:**
- **Pessimistic** → lock on read, hold until done. Safe but slow.
- **Optimistic** → no lock on read, check version on write. Fast but may need a retry on conflict.

---

## Database Isolation Levels

Isolation levels define what dirty/phantom reads are allowed. Higher isolation = safer but slower.

| Level | Dirty Read | Non-repeatable Read | Phantom Read |
|---|---|---|---|
| **Read Uncommitted** |  Possible | Possible |  Possible |
| **Read Committed** |  Not possible |  Possible |  Possible |
| **Repeatable Read** | Not possible| Not possible | Possible |
| **Serializable** | Not possible | Not possible | Not possible |

**Definitions:**
- **Dirty Read** → reading uncommitted data from another transaction (could be rolled back)
- **Non-repeatable Read** → same row, same query, different result in the same transaction (someone updated it)
- **Phantom Read** → same query, different rows returned (someone inserted/deleted rows)

```sql
-- Set isolation level in PostgreSQL
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- MySQL
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;
```

> I use **Read Committed** by default (PostgreSQL default). **Serializable** only when I need full correctness (financial systems). **Read Uncommitted** — almost never.

---

## Triggers

A trigger automatically fires when a specific event happens on a table.

```sql
-- Syntax (PostgreSQL)
CREATE OR REPLACE FUNCTION log_update()
RETURNS TRIGGER AS $$
BEGIN
  INSERT INTO audit_log(table_name, action, changed_at)
  VALUES ('employees', TG_OP, NOW());
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_employee_update
AFTER UPDATE ON employees
FOR EACH ROW
EXECUTE FUNCTION log_update();
```

### Trigger types I use

| Type | When it fires |
|---|---|
| `BEFORE INSERT/UPDATE/DELETE` | Before the operation. Can modify or cancel it. |
| `AFTER INSERT/UPDATE/DELETE` | After the operation. Good for audit logs. |
| `INSTEAD OF` | On views. Intercepts the operation entirely. |

### Special variables inside triggers

| Variable | Meaning |
|---|---|
| `NEW` | The new row (available in INSERT, UPDATE) |
| `OLD` | The old row (available in UPDATE, DELETE) |
| `TG_OP` | The operation: `INSERT`, `UPDATE`, `DELETE` |
| `TG_TABLE_NAME` | Name of the table that fired the trigger |

```sql
-- Example: prevent salary from going below 0
CREATE OR REPLACE FUNCTION check_salary()
RETURNS TRIGGER AS $$
BEGIN
  IF NEW.salary < 0 THEN
    RAISE EXCEPTION 'Salary cannot be negative';
  END IF;
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_salary_check
BEFORE UPDATE ON employees
FOR EACH ROW
EXECUTE FUNCTION check_salary();
```

---

# 🗄️ Part 4: Database & SQL (Q80–Q103)

---

## Q80. Database Locks

**Database locks** control concurrent access to data to maintain ACID properties.

| Lock Type | Description |
|-----------|-------------|
| **Shared Lock (S)** | Multiple transactions can READ, no one can WRITE |
| **Exclusive Lock (X)** | Only one transaction can READ/WRITE |
| **Row-level Lock** | Locks single row — high concurrency |
| **Table-level Lock** | Locks entire table — simple but blocks all |
| **Optimistic Lock** | No locking; checks version on commit (good for read-heavy) |
| **Pessimistic Lock** | Locks upfront; prevents conflicts (good for write-heavy) |

```sql
-- Row-level lock (SELECT FOR UPDATE)
BEGIN TRANSACTION;
SELECT * FROM accounts WHERE id = 101 FOR UPDATE;  -- locks row 101
UPDATE accounts SET balance = balance - 500 WHERE id = 101;
COMMIT;

-- Optimistic locking (version column)
UPDATE products SET stock = stock-1, version = version+1
WHERE id = 5 AND version = 3;  -- fails if version changed
```

---

## Q81. Truncate vs Delete vs Drop

| Feature | DELETE | TRUNCATE | DROP |
|---------|--------|----------|------|
| Type | DML | DDL | DDL |
| WHERE clause | ✅ Yes | ❌ No | ❌ No |
| Rows affected | Specific or all | All | N/A (removes table) |
| Rollback | ✅ Yes | ❌ No (auto-commit) | ❌ No |
| Triggers | ✅ Fires | ❌ Doesn't fire | ❌ N/A |
| Speed | Slow (row by row, logged) | Fast (deallocates pages) | Fastest |
| Space | Doesn't release | Releases | Releases everything |
| Identity/Auto-inc | NOT reset | Reset to seed | N/A |

```sql
DELETE FROM employees WHERE dept = 'HR';  -- selective, logged, can rollback
TRUNCATE TABLE employees;                  -- removes ALL rows, resets identity
DROP TABLE employees;                      -- table gone completely
```

> "Think of it as: DELETE = erasing with pencil (can undo), TRUNCATE = tearing all pages (keep notebook), DROP = throwing away the notebook."

---

## Q82. Views & Hibernate Queries on Views

A **View** is a virtual table based on a SQL query. No data stored — calculated on access.

```sql
CREATE VIEW high_salary_emp AS
SELECT name, salary, department FROM employees WHERE salary > 50000;

SELECT * FROM high_salary_emp;  -- treated like a table
```

**Types:** Simple View (single table, updatable), Complex View (joins, aggregates, read-only), Materialized View (stores data physically, refreshed periodically).

**Hibernate on Views:** Yes! Map a view like an entity:
```java
@Entity
@Table(name = "high_salary_emp")
@Immutable  // views are read-only
public class HighSalaryEmployee {
    @Id private Long id;
    private String name;
    private double salary;
}
```

---

## Q83. SQL Joins

```
INNER JOIN          LEFT JOIN           RIGHT JOIN          FULL OUTER JOIN
  A ∩ B             A + (A ∩ B)         (A ∩ B) + B         A ∪ B
  ┌───┐             ┌───────┐           ┌───────┐           ┌───────────┐
  │ ∩ │             │ A │ ∩ │           │ ∩ │ B │           │ A │ ∩ │ B │
  └───┘             └───────┘           └───────┘           └───────────┘
```

```sql
-- Sample: employees(id, name, dept_id) + departments(id, dept_name)

-- INNER JOIN — only matching rows
SELECT e.name, d.dept_name FROM employees e
INNER JOIN departments d ON e.dept_id = d.id;

-- LEFT JOIN — all from left + matching from right
SELECT e.name, d.dept_name FROM employees e
LEFT JOIN departments d ON e.dept_id = d.id;
-- Employees without department show NULL for dept_name

-- RIGHT JOIN — all from right + matching from left
SELECT e.name, d.dept_name FROM employees e
RIGHT JOIN departments d ON e.dept_id = d.id;

-- FULL OUTER JOIN — all from both
SELECT e.name, d.dept_name FROM employees e
FULL OUTER JOIN departments d ON e.dept_id = d.id;

-- CROSS JOIN — cartesian product (every combination)
SELECT e.name, d.dept_name FROM employees e CROSS JOIN departments d;

-- SELF JOIN — table joined with itself
SELECT e1.name AS employee, e2.name AS manager
FROM employees e1 JOIN employees e2 ON e1.manager_id = e2.id;
```

---

## Q84. UNION vs UNION ALL

| UNION | UNION ALL |
|-------|-----------|
| Removes duplicates | Keeps ALL rows including duplicates |
| Slower (sorts to remove dups) | Faster |

```sql
SELECT city FROM customers UNION SELECT city FROM suppliers;       -- unique cities
SELECT city FROM customers UNION ALL SELECT city FROM suppliers;   -- all cities (with dups)
```

---

## Q85. Triggers with Syntax

A **Trigger** is a stored procedure that runs automatically on INSERT/UPDATE/DELETE.

```sql
-- Audit trail trigger
CREATE TRIGGER trg_salary_audit
AFTER UPDATE OF salary ON employees
FOR EACH ROW
BEGIN
    INSERT INTO salary_audit (emp_id, old_salary, new_salary, changed_at)
    VALUES (:OLD.id, :OLD.salary, :NEW.salary, SYSDATE);
END;

-- Prevent deletion trigger
CREATE TRIGGER trg_prevent_delete
BEFORE DELETE ON employees
FOR EACH ROW
BEGIN
    RAISE_APPLICATION_ERROR(-20001, 'Cannot delete employees!');
END;
```

---

## Q86. Procedure vs Function vs Stored Procedure

| Feature | Function | Procedure |
|---------|----------|-----------|
| Return value | MUST return a value | May or may not (OUT params) |
| Call from SQL | ✅ Yes (SELECT, WHERE) | ❌ No |
| DML allowed | Usually No | ✅ Yes |
| Usage | Calculations | Business logic, DML operations |

```sql
-- FUNCTION
CREATE FUNCTION get_tax(salary NUMBER) RETURN NUMBER IS
BEGIN
    RETURN salary * 0.30;
END;
SELECT name, get_tax(salary) FROM employees;

-- STORED PROCEDURE
CREATE PROCEDURE update_salary(p_id IN NUMBER, p_hike IN NUMBER) IS
BEGIN
    UPDATE employees SET salary = salary + p_hike WHERE id = p_id;
    COMMIT;
END;
EXEC update_salary(101, 5000);
```

---

## Q87. What is a Cursor?

A **cursor** is a pointer to a result set, used to process rows **one by one**.

```sql
DECLARE
    CURSOR emp_cursor IS SELECT id, name, salary FROM employees WHERE dept='IT';
    v_id employees.id%TYPE;
    v_name employees.name%TYPE;
    v_salary employees.salary%TYPE;
BEGIN
    OPEN emp_cursor;
    LOOP
        FETCH emp_cursor INTO v_id, v_name, v_salary;
        EXIT WHEN emp_cursor%NOTFOUND;
        DBMS_OUTPUT.PUT_LINE(v_name || ' earns ' || v_salary);
    END LOOP;
    CLOSE emp_cursor;
END;
```

**Types:** Implicit (single-row queries, auto-managed), Explicit (multi-row, manual control).

---

## Q88. Primary Key vs Foreign Key vs Unique Constraint

| Feature | Primary Key | Foreign Key | Unique Constraint |
|---------|------------|-------------|-------------------|
| Purpose | Uniquely identifies row | References PK in another table | Ensures unique values |
| NULL? | ❌ NOT NULL | ✅ Can be NULL | ✅ One NULL allowed |
| Per table | Only ONE | Multiple | Multiple |
| Index | Clustered auto | No auto index | Non-clustered auto |

```sql
CREATE TABLE departments (
    dept_id INT PRIMARY KEY,           -- PK
    dept_name VARCHAR(50) UNIQUE       -- Unique
);

CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    email VARCHAR(100) UNIQUE,
    dept_id INT,
    FOREIGN KEY (dept_id) REFERENCES departments(dept_id)  -- FK
        ON DELETE CASCADE ON UPDATE CASCADE
);
```

---

## Q89. Normalization, 1NF, 2NF

**Normalization** = organizing data to reduce redundancy.

**1NF:** Each column has atomic (indivisible) values. No repeating groups.
```
❌ Violates 1NF: Student | Phones: "123, 456, 789"
✅ 1NF: Each phone in separate row or separate table
```

**2NF:** 1NF + no partial dependency (every non-key depends on ENTIRE primary key).
```
❌ Violates 2NF: (StudentID, CourseID) → StudentName  (depends only on StudentID)
✅ 2NF: Move StudentName to Students table
```

**3NF:** 2NF + no transitive dependency (non-key depends only on PK, not on other non-keys).

---

## Q90. Indexes and Types

**Index** = data structure (B-Tree/Hash) that speeds up SELECT but slows INSERT/UPDATE/DELETE.

```sql
CREATE INDEX idx_emp_name ON employees(name);           -- Single-column
CREATE INDEX idx_dept_salary ON employees(dept, salary); -- Composite
CREATE UNIQUE INDEX idx_email ON employees(email);       -- Unique
```

| Type | Description |
|------|-------------|
| Clustered | Sorts actual data (1 per table — usually PK) |
| Non-clustered | Separate structure with pointers |
| Composite | Multiple columns |
| Unique | Enforces uniqueness |
| Full-text | For text search |

**When index FAILS:** Using functions on indexed column (`WHERE UPPER(name)='JOHN'`), using `LIKE '%pattern'` (leading wildcard), low cardinality columns, small tables.

---

## Q91. SQL Injection with Example and Solution

**Attack:** Malicious SQL code injected through user input.

```java
// VULNERABLE ❌
String query = "SELECT * FROM users WHERE username='" + input + "' AND password='" + pass + "'";
// Input: username = ' OR '1'='1' --
// Becomes: SELECT * FROM users WHERE username='' OR '1'='1' --' AND password=''
// → Returns ALL users! Attacker logs in!

// SOLUTION ✅ — Prepared Statements (Parameterized Queries)
PreparedStatement ps = conn.prepareStatement("SELECT * FROM users WHERE username=? AND password=?");
ps.setString(1, username);  // safely escaped
ps.setString(2, password);
ResultSet rs = ps.executeQuery();
```

**Additional defenses:** Input validation, ORM (Hibernate), stored procedures, least privilege, WAF.

---

## Q92-Q93. Pseudo Columns & GROUP BY with aggregate functions

**Pseudo columns** — columns not part of table but available in queries:
- `ROWNUM` — row number in result set
- `ROWID` — physical address of row
- `SYSDATE` — current date/time
- `USER` — current user

**GROUP BY mandatory?** No — aggregate functions work without GROUP BY (treats entire table as one group):
```sql
SELECT COUNT(*), AVG(salary), MAX(salary) FROM employees;  -- no GROUP BY needed
SELECT dept, AVG(salary) FROM employees GROUP BY dept;       -- GROUP BY needed here
```

---

## Q94. 2nd Highest Salary (Multiple Ways)

```sql
-- Way 1: Subquery
SELECT MAX(salary) FROM employees WHERE salary < (SELECT MAX(salary) FROM employees);

-- Way 2: LIMIT/OFFSET
SELECT DISTINCT salary FROM employees ORDER BY salary DESC LIMIT 1 OFFSET 1;

-- Way 3: DENSE_RANK()
SELECT salary FROM (
    SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) AS rank FROM employees
) ranked WHERE rank = 2;

-- Way 4: NOT IN
SELECT MAX(salary) FROM employees WHERE salary NOT IN (SELECT MAX(salary) FROM employees);

-- Nth highest (generic):
SELECT salary FROM (
    SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) AS rnk FROM employees
) t WHERE rnk = N;
```

---

## Q95. Copy content of one table to another

```sql
-- Copy structure + data
SELECT * INTO new_table FROM old_table;

-- Copy only data (table exists)
INSERT INTO target_table SELECT * FROM source_table;

-- Copy with condition
INSERT INTO archive_employees SELECT * FROM employees WHERE resign_date IS NOT NULL;

-- Copy structure only (no data)
SELECT * INTO new_table FROM old_table WHERE 1 = 0;
```

---

## Q96. WHERE vs HAVING

| WHERE | HAVING |
|-------|--------|
| Filters rows BEFORE grouping | Filters groups AFTER grouping |
| Cannot use aggregate functions | CAN use aggregate functions |
| Works on individual rows | Works on grouped results |

```sql
-- WHERE: filter rows
SELECT * FROM employees WHERE salary > 50000;

-- HAVING: filter groups
SELECT dept, AVG(salary) AS avg_sal
FROM employees
WHERE status = 'ACTIVE'     -- WHERE filters rows first
GROUP BY dept
HAVING AVG(salary) > 60000; -- HAVING filters groups after
```

---

## Q97-Q103. Normalization vs Denormalization, Isolation Levels, Deadlocks, Composite Key, Transactions

**Normalization vs Denormalization:**

| Normalization | Denormalization |
|---------------|-----------------|
| Remove redundancy | Add redundancy intentionally |
| More tables, more joins | Fewer joins, faster reads |
| Better write performance | Better read performance |
| Data integrity | Data may be inconsistent |
| OLTP systems | OLAP/Reporting systems |

**Isolation Levels (low → high):**

| Level | Dirty Read | Non-Repeatable Read | Phantom Read |
|-------|-----------|-------------------|-------------|
| READ UNCOMMITTED | ✅ | ✅ | ✅ |
| READ COMMITTED | ❌ | ✅ | ✅ |
| REPEATABLE READ | ❌ | ❌ | ✅ |
| SERIALIZABLE | ❌ | ❌ | ❌ |

**DB Deadlock:** Two transactions waiting for each other's locks. DB detects and kills one (victim) with rollback.

**Composite Key:** Primary key with multiple columns: `PRIMARY KEY (student_id, course_id)`

**Foreign Key Constraint Behaviors:**
- `ON DELETE CASCADE` — delete child rows
- `ON DELETE SET NULL` — set FK to null
- `ON DELETE RESTRICT` — prevent parent deletion

**Transaction Example:**
```sql
BEGIN TRANSACTION;
UPDATE accounts SET balance = balance - 500 WHERE id = 1;
UPDATE accounts SET balance = balance + 500 WHERE id = 2;
-- If any fails:
ROLLBACK;
-- If all succeed:
COMMIT;
```

**Index Disadvantages:** Extra storage, slower writes (INSERT/UPDATE/DELETE must update index), maintenance overhead for frequent updates.

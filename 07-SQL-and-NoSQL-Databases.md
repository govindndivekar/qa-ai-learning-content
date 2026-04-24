# SQL and NoSQL Databases — From Fundamentals to Expert, with DB Testing

> Subject #7 of the Master Learning Plan.
> Owner: Govind Divekar. Prerequisites: None. Estimated time: ~70 hours.
> Target outcome: Design and query relational and NoSQL data stores, understand consistency and performance trade-offs, and test data layers professionally.

---

## 1. Overview & Learning Outcomes

This subject spans three major domains: **relational databases and SQL** (Part A), **NoSQL and polyglot persistence** (Part B), and **database testing & QA** (Part C). By the end, you will:

- Write performant SQL queries: joins, window functions, CTEs, subqueries, and explain plans.
- Model relational schemas using normalization and intentional denormalization patterns.
- Understand and query NoSQL engines: key-value (Redis), document (MongoDB), wide-column (DynamoDB), graph (Neo4j), time-series, search, and vector stores.
- Master database testing: migrations, data quality, performance profiling, and integration testing.
- Choose the right database for a given workload and detect common pitfalls.
- Confidently discuss CAP theorem, consistency models, and trade-offs in interviews.

---

## 2. Detailed Syllabus

**Part A: Relational & SQL (30 hours)**
- Topic 1-6: Postgres setup, relational model, data types, DDL, DML, SELECT basics (6 hours)
- Topic 7-11: Joins, grouping, subqueries, CTEs, window functions (10 hours)
- Topic 12-16: Views, indexes, EXPLAIN, transactions, locks (8 hours)
- Topic 17-22: Normalization, modeling patterns, JSON, PL/pgSQL, query tuning, security (6 hours)

**Part B: NoSQL (20 hours)**
- Topic 23-26: CAP/PACELC, Redis, MongoDB, Cassandra/DynamoDB (10 hours)
- Topic 27-32: DynamoDB specifics, Neo4j, time-series, Elasticsearch, vector DBs, polyglot design (10 hours)

**Part C: Database Testing & QA (20 hours)**
- Topic 33-37: Schema testing, migrations, seed/teardown, contracts (8 hours)
- Topic 38-43: Data quality, performance testing, replication, PII, NoSQL testing, interview Q&A (12 hours)

---

## 3. Topics — Explanations with Examples

### **PART A: RELATIONAL & SQL**

---

#### **Topic 1: Running Postgres Locally**

**Concept**: Postgres is a robust, open-source RDBMS. For development and learning, run it in Docker, then interact via `psql` (CLI), DBeaver, or TablePlus (GUI).

**Setup:**
```bash
# Pull and run Postgres in Docker
docker run --name postgres_dev -e POSTGRES_USER=dev -e POSTGRES_PASSWORD=dev123 \
  -e POSTGRES_DB=testdb -p 5432:5432 -d postgres:15

# Connect via psql (install locally or via Docker)
psql -h localhost -U dev -d testdb -W

# Within psql:
\dt                    # List tables
\d table_name          # Describe a table
\q                     # Quit
SELECT version();      # Check Postgres version
```

**Pitfalls:**
- Forgetting to expose the port (`-p 5432:5432`) → can't connect.
- Not setting `POSTGRES_PASSWORD` → random password generated.
- Mixing up `-U` (user) and `-d` (database).

---

#### **Topic 2: Relational Model**

**Concept**: A relation is a table; a tuple is a row. Keys uniquely identify tuples: primary keys (PK) enforce uniqueness and non-null; foreign keys (FK) reference other tables; unique keys allow one null.

**Example:**
```sql
-- Schema: authors and books
CREATE TABLE authors (
  author_id SERIAL PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  email VARCHAR(255) UNIQUE,
  birth_date DATE
);

CREATE TABLE books (
  book_id SERIAL PRIMARY KEY,
  title VARCHAR(500) NOT NULL,
  author_id INT NOT NULL REFERENCES authors(author_id),
  isbn VARCHAR(20) UNIQUE,
  published_year INT CHECK (published_year >= 1000),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**Pitfalls:**
- No PK = slow joins and no clear identity.
- Missing FK constraints = orphaned rows.
- Forgetting `NOT NULL` on required columns = silent nulls in queries.

---

#### **Topic 3: Data Types**

**Concept**: Choose the right type to save space, enforce domain rules, and enable index optimization.

**Common types:**
```sql
-- Numeric
INT, BIGINT, SMALLINT, DECIMAL(10,2), FLOAT, NUMERIC

-- Text
VARCHAR(n), TEXT, CHAR(n)

-- Boolean
BOOLEAN  -- values: TRUE, FALSE, NULL

-- Date/time
DATE, TIME, TIMESTAMP, TIMESTAMPTZ (timezone-aware)

-- Special
UUID, SERIAL (auto-increment), ARRAY, JSON/JSONB

-- Array example
CREATE TABLE orders (
  order_id SERIAL PRIMARY KEY,
  item_ids INT[] NOT NULL,           -- ARRAY[1,2,3]
  quantities DECIMAL[] DEFAULT '{}'
);

-- JSON example
CREATE TABLE users (
  user_id SERIAL PRIMARY KEY,
  metadata JSONB NOT NULL DEFAULT '{}'  -- JSONB for indexing
);
```

**Pitfalls:**
- Using TEXT for everything = slow comparisons and indexes.
- Using TIMESTAMP instead of TIMESTAMPTZ in multi-region apps = timezone chaos.
- SERIAL is not a true type; GENERATED ALWAYS AS IDENTITY is newer, safer.

---

#### **Topic 4: DDL (CREATE, ALTER, DROP)**

**Concept**: Define schema structure. DDL changes lock tables; avoid in production without planning.

**Examples:**
```sql
-- Create
CREATE TABLE IF NOT EXISTS products (
  product_id SERIAL PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  price DECIMAL(10, 2) CHECK (price > 0),
  category_id INT REFERENCES categories(category_id)
);

-- Alter: add column
ALTER TABLE products ADD COLUMN description TEXT DEFAULT '';

-- Alter: rename
ALTER TABLE products RENAME COLUMN name TO product_name;

-- Alter: add constraint
ALTER TABLE products ADD CONSTRAINT unique_product_name UNIQUE (product_name);

-- Drop (destructive!)
DROP TABLE IF EXISTS temp_table CASCADE;  -- CASCADE drops dependent objects

-- Schemas (namespaces)
CREATE SCHEMA ecommerce;
CREATE TABLE ecommerce.products (...);
```

**Pitfalls:**
- Adding NOT NULL without default to a non-empty table = error.
- DROP without IF EXISTS and CASCADE = failure if dependencies exist.
- Altering large tables in production = locks out readers; use `CONCURRENTLY` for indexes.

---

#### **Topic 5: DML (INSERT, UPDATE, DELETE, UPSERT)**

**Concept**: Modify data. Use RETURNING to confirm changes; UPSERT (ON CONFLICT) to merge.

**Examples:**
```sql
-- Insert
INSERT INTO authors (name, email) VALUES ('Jane Austen', 'jane@example.com');

-- Insert with RETURNING
INSERT INTO authors (name, email) 
VALUES ('Mark Twain', 'mark@example.com')
RETURNING author_id, name;

-- Bulk insert
INSERT INTO authors (name, email) VALUES
  ('Charlotte Brontë', 'charlotte@example.com'),
  ('Emily Dickinson', 'emily@example.com');

-- Update
UPDATE authors SET email = 'newemail@example.com' WHERE name = 'Jane Austen';

-- Update with RETURNING
UPDATE authors 
SET email = 'updated@example.com' 
WHERE author_id = 1
RETURNING *;

-- Delete
DELETE FROM books WHERE published_year < 1800;

-- Upsert (ON CONFLICT)
INSERT INTO authors (name, email) VALUES ('Jane Austen', 'jane.austen@example.com')
ON CONFLICT (email) DO UPDATE SET name = EXCLUDED.name;
```

**Pitfalls:**
- Forgetting WHERE in DELETE = wipes the table.
- Missing ON CONFLICT when IDs may repeat = duplicate key error.
- Not using RETURNING to verify the change.

---

#### **Topic 6: SELECT Fundamentals**

**Concept**: Retrieve, filter, and sort data. SELECT is the most common operation; master projection, filtering, sorting, LIMIT, OFFSET, DISTINCT.

**Examples:**
```sql
-- Projection: select specific columns
SELECT author_id, name FROM authors;

-- Filtering: WHERE clause
SELECT * FROM books WHERE published_year >= 2000 AND author_id = 1;

-- Sorting: ORDER BY
SELECT * FROM books ORDER BY published_year DESC, title ASC;

-- Limit and offset: pagination
SELECT * FROM books ORDER BY published_year DESC LIMIT 10 OFFSET 20;

-- Distinct: remove duplicates
SELECT DISTINCT category_id FROM products;

-- Aliases
SELECT author_id AS id, name AS author_name FROM authors;

-- String operations
SELECT * FROM authors WHERE name ILIKE '%austen%';  -- case-insensitive

-- IN clause
SELECT * FROM books WHERE author_id IN (1, 3, 5);

-- BETWEEN
SELECT * FROM orders WHERE order_date BETWEEN '2024-01-01' AND '2024-12-31';
```

**Pitfalls:**
- Using `=` with NULL = always FALSE; use `IS NULL` instead.
- OFFSET without ORDER BY = non-deterministic results.
- No LIMIT on large tables = memory explosion.

---

#### **Topic 7: Joins**

**Concept**: Combine rows from multiple tables. INNER JOIN returns matching rows; LEFT/RIGHT/FULL OUTER include non-matches; CROSS JOIN is a Cartesian product.

**Examples:**
```sql
-- INNER JOIN
SELECT a.name, b.title 
FROM authors a 
INNER JOIN books b ON a.author_id = b.author_id;

-- LEFT JOIN (all authors, with books if they exist)
SELECT a.name, COUNT(b.book_id) as book_count
FROM authors a
LEFT JOIN books b ON a.author_id = b.author_id
GROUP BY a.author_id, a.name;

-- RIGHT JOIN (all books, with author info)
SELECT b.title, a.name
FROM authors a
RIGHT JOIN books b ON a.author_id = b.author_id;

-- FULL OUTER JOIN (all rows from both sides)
SELECT a.name, b.title
FROM authors a
FULL OUTER JOIN books b ON a.author_id = b.author_id;

-- CROSS JOIN (Cartesian product: every row paired with every row)
SELECT a.name, c.category
FROM authors a
CROSS JOIN (SELECT 'Fiction' as category UNION SELECT 'Non-Fiction') c;

-- Self-join (compare within same table)
SELECT e1.name, e2.name
FROM employees e1
JOIN employees e2 ON e1.manager_id = e2.employee_id;

-- LATERAL join (each row can see previous rows)
SELECT a.name, b_info.book_count
FROM authors a
LATERAL (
  SELECT COUNT(*) as book_count FROM books WHERE author_id = a.author_id
) b_info;
```

**Pitfalls:**
- Forgetting the ON condition = CROSS JOIN (huge result set).
- Using WHERE instead of ON in LEFT JOIN = filters out nulls, defeating the join.
- Multiple LEFT JOINs without aggregation = row explosion (n * m * k rows).

---

#### **Topic 8: Grouping (GROUP BY, HAVING, GROUPING SETS, ROLLUP, CUBE)**

**Concept**: Aggregate rows by a dimension. GROUP BY partitions; aggregate functions (SUM, COUNT, AVG, MIN, MAX) combine values. HAVING filters groups.

**Examples:**
```sql
-- Basic GROUP BY
SELECT author_id, COUNT(*) as num_books 
FROM books 
GROUP BY author_id;

-- GROUP BY with HAVING
SELECT author_id, COUNT(*) as num_books 
FROM books 
GROUP BY author_id 
HAVING COUNT(*) > 2;

-- Multiple columns
SELECT category_id, year, COUNT(*) as sales_count
FROM sales
GROUP BY category_id, year;

-- GROUPING SETS (compute multiple groupings)
SELECT category_id, year, COUNT(*) 
FROM sales
GROUP BY GROUPING SETS (
  (category_id, year),
  (category_id),
  (year),
  ()
);

-- ROLLUP (hierarchical aggregation)
SELECT year, quarter, month, SUM(revenue)
FROM sales
GROUP BY ROLLUP (year, quarter, month);

-- CUBE (all combinations)
SELECT year, category, COUNT(*)
FROM sales
GROUP BY CUBE (year, category);
```

**Pitfalls:**
- Non-aggregated columns in SELECT without GROUP BY = error or non-deterministic row.
- Using WHERE instead of HAVING to filter on aggregates = wrong (WHERE filters before grouping).
- Forgetting to list all non-aggregate columns in GROUP BY.

---

#### **Topic 9: Subqueries**

**Concept**: Nest queries. Scalar subqueries return one value; correlated subqueries reference outer rows; EXISTS/IN are filtering subqueries.

**Examples:**
```sql
-- Scalar subquery (single value)
SELECT name, (SELECT COUNT(*) FROM books WHERE author_id = authors.author_id) as book_count
FROM authors;

-- Subquery in WHERE
SELECT * FROM books WHERE author_id IN (SELECT author_id FROM authors WHERE birth_year > 1800);

-- EXISTS (more efficient than IN for large datasets)
SELECT * FROM authors a WHERE EXISTS (SELECT 1 FROM books b WHERE b.author_id = a.author_id);

-- NOT EXISTS
SELECT * FROM authors a WHERE NOT EXISTS (SELECT 1 FROM books b WHERE b.author_id = a.author_id);

-- Correlated subquery
SELECT a.name, 
  (SELECT MAX(published_year) FROM books b WHERE b.author_id = a.author_id) as latest_book_year
FROM authors a;

-- Subquery in FROM (derived table; must be aliased)
SELECT avg_price FROM (
  SELECT AVG(price) as avg_price FROM products
) derived;
```

**Pitfalls:**
- IN with NULL in subquery = NULL results (use NOT IN with NULL = no matches).
- Correlated subqueries run per outer row = O(n) queries; use JOIN instead.
- Missing alias on derived table = error.

---

#### **Topic 10: CTEs (Common Table Expressions) & Recursive CTEs**

**Concept**: WITH clause defines temporary named result sets. Recursive CTEs loop (base case + recursive union).

**Examples:**
```sql
-- Simple CTE
WITH author_books AS (
  SELECT author_id, COUNT(*) as book_count 
  FROM books 
  GROUP BY author_id
)
SELECT a.name, ab.book_count
FROM authors a
JOIN author_books ab ON a.author_id = ab.author_id;

-- Multiple CTEs
WITH active_authors AS (
  SELECT author_id FROM authors WHERE death_date IS NULL
),
recent_books AS (
  SELECT author_id, COUNT(*) as count FROM books WHERE published_year >= 2020 GROUP BY author_id
)
SELECT aa.author_id, rb.count FROM active_authors aa
LEFT JOIN recent_books rb ON aa.author_id = rb.author_id;

-- Recursive CTE (tree traversal)
WITH RECURSIVE category_tree AS (
  -- Base case
  SELECT category_id, parent_id, name, 1 as depth
  FROM categories
  WHERE parent_id IS NULL
  
  UNION ALL
  
  -- Recursive case
  SELECT c.category_id, c.parent_id, ct.name || ' > ' || c.name, ct.depth + 1
  FROM categories c
  JOIN category_tree ct ON c.parent_id = ct.category_id
  WHERE ct.depth < 10  -- prevent infinite loops
)
SELECT * FROM category_tree;
```

**Pitfalls:**
- Recursive CTE without depth limit or termination = infinite loop.
- CTE is evaluated once per query; use them to improve readability, not as views.
- Complex nesting = hard to debug; start simple.

---

#### **Topic 11: Window Functions**

**Concept**: Apply an aggregate or ranking over a "window" of rows (PARTITION BY + ORDER BY). No grouping; retains all rows.

**Examples:**
```sql
-- ROW_NUMBER (unique rank even with ties)
SELECT employee_id, salary,
  ROW_NUMBER() OVER (ORDER BY salary DESC) as rank
FROM employees;

-- RANK (same rank for ties, skips numbers)
SELECT employee_id, salary,
  RANK() OVER (ORDER BY salary DESC) as rank
FROM employees;

-- DENSE_RANK (same rank for ties, no skips)
SELECT employee_id, salary,
  DENSE_RANK() OVER (ORDER BY salary DESC) as rank
FROM employees;

-- LAG / LEAD (look back/forward)
SELECT order_id, order_date, amount,
  LAG(amount) OVER (ORDER BY order_date) as prev_amount,
  LEAD(amount) OVER (ORDER BY order_date) as next_amount
FROM orders;

-- Running total with ROWS frame
SELECT order_id, amount,
  SUM(amount) OVER (ORDER BY order_date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) as running_total
FROM orders;

-- Top N per group (PARTITION BY)
SELECT product_id, category_id, sales,
  ROW_NUMBER() OVER (PARTITION BY category_id ORDER BY sales DESC) as rank_in_category
FROM products
WHERE ROW_NUMBER() OVER (PARTITION BY category_id ORDER BY sales DESC) <= 5;  -- Won't work; use CTE

-- Top N per group (correct: use CTE)
WITH ranked AS (
  SELECT product_id, category_id, sales,
    ROW_NUMBER() OVER (PARTITION BY category_id ORDER BY sales DESC) as rn
  FROM products
)
SELECT * FROM ranked WHERE rn <= 5;

-- NTILE (divide into quartiles)
SELECT employee_id, salary,
  NTILE(4) OVER (ORDER BY salary) as quartile
FROM employees;
```

**Pitfalls:**
- PARTITION BY all rows = single window (defeats purpose).
- Forgetting ORDER BY in window function = non-deterministic order.
- Filtering window function results in WHERE = doesn't work; use CTE or subquery.
- ROWS vs RANGE: ROWS counts physical rows; RANGE counts logical values (use ROWS for most cases).

---

#### **Topic 12: Views & Materialized Views**

**Concept**: Views are virtual tables (queries stored); materialized views cache results to disk.

**Examples:**
```sql
-- Simple view
CREATE VIEW author_book_counts AS
SELECT a.author_id, a.name, COUNT(b.book_id) as book_count
FROM authors a
LEFT JOIN books b ON a.author_id = b.author_id
GROUP BY a.author_id, a.name;

SELECT * FROM author_book_counts WHERE book_count > 5;

-- Updatable view (must be based on single table, no aggregation)
CREATE VIEW simple_books AS
SELECT book_id, title, author_id FROM books;

UPDATE simple_books SET title = 'New Title' WHERE book_id = 1;  -- Works

-- Materialized view (physical copy; must refresh)
CREATE MATERIALIZED VIEW mv_author_book_counts AS
SELECT a.author_id, a.name, COUNT(b.book_id) as book_count
FROM authors a
LEFT JOIN books b ON a.author_id = b.author_id
GROUP BY a.author_id, a.name;

-- Refresh
REFRESH MATERIALIZED VIEW mv_author_book_counts;
REFRESH MATERIALIZED VIEW CONCURRENTLY mv_author_book_counts;  -- Allow concurrent reads

-- Drop
DROP VIEW IF EXISTS author_book_counts CASCADE;
DROP MATERIALIZED VIEW IF EXISTS mv_author_book_counts;
```

**Pitfalls:**
- Views don't improve performance (they're just rewritten queries).
- Materialized views go stale; refresh strategy critical.
- Dropping a view cascades if other objects depend on it.

---

#### **Topic 13: Indexes**

**Concept**: Speed up searches. B-tree (default) for range queries; hash for equality; GIN/GIST for complex types; BRIN for large sorted data. Covering indexes avoid table lookups.

**Examples:**
```sql
-- B-tree (default; supports <, >, <=, >=, =)
CREATE INDEX idx_books_author_id ON books(author_id);
CREATE INDEX idx_books_year_author ON books(published_year DESC, author_id);  -- Composite

-- Unique index
CREATE UNIQUE INDEX idx_authors_email ON authors(email);

-- Partial index (smaller, faster)
CREATE INDEX idx_active_users ON users(user_id) WHERE is_active = true;

-- Expression index (index computed value)
CREATE INDEX idx_authors_name_lower ON authors(LOWER(name));

-- Hash index (fast equality only)
CREATE INDEX idx_users_email_hash ON users USING HASH(email);

-- GIN index (good for arrays, JSON, full-text)
CREATE INDEX idx_orders_items ON orders USING GIN(item_ids);
CREATE INDEX idx_users_tags ON users USING GIN(tags);

-- Covering index (include non-key columns to avoid table lookup)
CREATE INDEX idx_books_covering ON books(author_id) INCLUDE (title, published_year);

-- BRIN index (for large, naturally sorted data; tiny size)
CREATE INDEX idx_events_timestamp ON events USING BRIN(event_timestamp);

-- Full-text search index
CREATE INDEX idx_articles_search ON articles USING GIN(to_tsvector('english', content));

-- List all indexes
SELECT * FROM pg_indexes WHERE tablename = 'books';
DROP INDEX IF EXISTS idx_books_author_id;
```

**Pitfalls:**
- Indexes speed up reads but slow down writes (every INSERT/UPDATE must update indexes).
- Too many indexes = maintenance nightmare and memory bloat.
- Indexes not used if query filter is non-indexed or uses function without expression index.
- LIKE '%pattern' = full table scan (no index); '%pattern%' especially bad.

---

#### **Topic 14: EXPLAIN & Query Plans**

**Concept**: EXPLAIN shows the query plan; EXPLAIN ANALYZE runs it and shows actual timings. Understand sequential scan vs index scan.

**Examples:**
```sql
-- Show plan (no execution)
EXPLAIN SELECT * FROM books WHERE author_id = 5;

-- Show plan with actual runtime stats
EXPLAIN ANALYZE SELECT * FROM books WHERE author_id = 5;

-- Verbose output
EXPLAIN (ANALYZE, VERBOSE, BUFFERS) SELECT * FROM books WHERE author_id = 5;

-- Example output interpretation:
-- Seq Scan on books  (cost=0.00..35.50 rows=10 width=100)
--   Filter: author_id = 5
--   Planning Time: 0.05 ms
--   Execution Time: 0.15 ms

-- "cost" = arbitrary units (startup..total)
-- "rows" = estimated rows (vs actual after ANALYZE)
-- "width" = average row bytes
-- Seq Scan = table scan; Index Scan = uses index
-- Index Cond = filter applied by index; Filter = applied after

-- Common plan nodes:
-- - Seq Scan: full table scan
-- - Index Scan: using index
-- - Bitmap Index Scan: index with additional filter
-- - Hash Join: join using hash table (good for large sets)
-- - Nested Loop: join comparing every pair (O(n*m); slow for large)
-- - Sort: expensive; check ORDER BY necessity
-- - Limit: stops early; fast
```

**Pitfalls:**
- Estimated rows far off from actual = stale statistics; run ANALYZE.
- Seq Scan on large table = likely missing index.
- Nested Loop on two large tables = slow join; add index on join column.
- N+1 queries (one outer, n inner) = hidden sequential loop; use JOIN or batch.

---

#### **Topic 15: Transactions & ACID**

**Concept**: ACID ensures data integrity. Atomicity = all-or-nothing; Consistency = valid state; Isolation = no interference; Durability = survives crashes. Isolation levels trade off consistency vs performance.

**Examples:**
```sql
-- Basic transaction
BEGIN;
INSERT INTO accounts (name, balance) VALUES ('Alice', 1000);
UPDATE accounts SET balance = balance - 100 WHERE name = 'Alice';
COMMIT;

-- Rollback on error
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE name = 'Alice';
-- Oops, wrong account!
ROLLBACK;

-- Savepoint (nested rollback)
BEGIN;
INSERT INTO accounts VALUES ('Bob', 500);
SAVEPOINT sp1;
INSERT INTO transactions VALUES ('transfer', -100);
-- Error detected
ROLLBACK TO sp1;
COMMIT;  -- Bob created, but transfer rolled back

-- Isolation levels (default: READ COMMITTED)
BEGIN ISOLATION LEVEL READ UNCOMMITTED;  -- Dirty reads possible
SELECT * FROM accounts;
COMMIT;

BEGIN ISOLATION LEVEL READ COMMITTED;   -- No dirty reads, phantom reads possible
SELECT * FROM accounts WHERE balance > 100;
COMMIT;

BEGIN ISOLATION LEVEL REPEATABLE READ;  -- No phantom reads (MySQL), but Postgres still allows them
SELECT COUNT(*) FROM accounts;
COMMIT;

BEGIN ISOLATION LEVEL SERIALIZABLE;     -- Fully isolated; slowest
SELECT * FROM accounts;
COMMIT;
```

**Pitfalls:**
- Assuming no anomalies without proper isolation level.
- Long-running transactions hold locks = blocks other users.
- Not handling ROLLBACK in application code = corrupted state.
- Isolation level too high (SERIALIZABLE) = serialization conflicts.

---

#### **Topic 16: Locks & Deadlocks**

**Concept**: Locks prevent concurrent modification. Row locks for specific rows; table locks for whole table. Deadlock occurs when two transactions wait on each other.

**Examples:**
```sql
-- SELECT FOR UPDATE (row lock until transaction ends)
BEGIN;
SELECT * FROM accounts WHERE account_id = 1 FOR UPDATE;
-- Other transactions blocked from updating this row
UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
COMMIT;

-- SELECT FOR SHARE (read lock; multiple readers, no writers)
BEGIN;
SELECT * FROM accounts WHERE account_id = 1 FOR SHARE;
SELECT * FROM accounts WHERE account_id = 1 FOR SHARE;  -- OK
COMMIT;

-- Advisory locks (application-level)
SELECT pg_advisory_lock(1);  -- Acquire lock
SELECT pg_advisory_unlock(1);  -- Release

-- Detect locks
SELECT * FROM pg_locks;
SELECT * FROM pg_stat_activity WHERE state = 'active';

-- Kill a long-running query
SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE pid = 12345;

-- Deadlock example (two transactions, opposite order):
-- Transaction A: UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
--                UPDATE accounts SET balance = balance + 100 WHERE account_id = 2;
-- Transaction B: UPDATE accounts SET balance = balance - 100 WHERE account_id = 2;
--                UPDATE accounts SET balance = balance + 100 WHERE account_id = 1;
-- Result: deadlock detected, one transaction aborted

-- Prevent deadlock: always lock in same order
```

**Pitfalls:**
- Forgetting FOR UPDATE in concurrent financial transactions = double-spending.
- Acquiring locks in different orders = deadlock risk.
- Long lock holds = blocks users; keep transactions short.

---

#### **Topic 17: Normalization & Denormalization**

**Concept**: Normalization (1NF → BCNF) removes redundancy and anomalies. Denormalization reintroduces redundancy for performance (careful!).

**Examples:**
```sql
-- 1NF: atomic values, no repeating groups
-- Bad: student_id, name, courses (CSV)
-- Good:
CREATE TABLE students (
  student_id SERIAL PRIMARY KEY,
  name VARCHAR(255)
);
CREATE TABLE enrollments (
  enrollment_id SERIAL PRIMARY KEY,
  student_id INT REFERENCES students,
  course_id INT REFERENCES courses
);

-- 2NF: remove partial dependencies (non-key columns depend on part of composite key)
-- Bad: order_id, product_id, warehouse_id, warehouse_location (location depends on warehouse, not order)
-- Good:
CREATE TABLE warehouses (
  warehouse_id SERIAL PRIMARY KEY,
  location VARCHAR(255)
);
CREATE TABLE order_items (
  order_id INT, product_id INT,
  warehouse_id INT REFERENCES warehouses,
  PRIMARY KEY (order_id, product_id),
  quantity INT
);

-- 3NF: remove transitive dependencies (non-key columns depend on other non-key columns)
-- Bad: order_id, customer_id, customer_city (city depends on customer, not order)
-- Good:
CREATE TABLE customers (
  customer_id SERIAL PRIMARY KEY,
  city VARCHAR(255)
);
CREATE TABLE orders (
  order_id SERIAL PRIMARY KEY,
  customer_id INT REFERENCES customers
);

-- BCNF: even stricter (all determinants are candidate keys)
-- Usually achieved after 3NF

-- Intentional denormalization: store computed value for performance
CREATE TABLE orders (
  order_id SERIAL PRIMARY KEY,
  customer_id INT,
  total DECIMAL(10, 2),  -- Denormalized; computed from order_items
  updated_at TIMESTAMP
);
-- Trade-off: faster reads, slower writes, risk of staleness
```

**Pitfalls:**
- Over-normalizing = excessive JOINs = slow queries.
- Under-normalizing = redundancy = anomalies.
- Denormalization without a clear performance need = maintenance burden.

---

#### **Topic 18: Modeling Patterns**

**Concept**: Common structural patterns for real-world problems.

**Examples:**
```sql
-- Lookup table (many-to-one mapping)
CREATE TABLE statuses (
  status_id SMALLINT PRIMARY KEY,
  name VARCHAR(50)
);
INSERT INTO statuses VALUES (1, 'pending'), (2, 'approved'), (3, 'rejected');

CREATE TABLE requests (
  request_id SERIAL PRIMARY KEY,
  status_id SMALLINT REFERENCES statuses
);

-- Audit table (track changes)
CREATE TABLE employees (
  employee_id SERIAL PRIMARY KEY,
  name VARCHAR(255),
  salary DECIMAL(10, 2),
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE employee_audit (
  audit_id SERIAL PRIMARY KEY,
  employee_id INT,
  old_salary DECIMAL(10, 2),
  new_salary DECIMAL(10, 2),
  changed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Trigger for audit (simplified)
-- CREATE TRIGGER audit_salary AFTER UPDATE ON employees ...

-- Soft delete (mark as deleted, don't remove)
CREATE TABLE products (
  product_id SERIAL PRIMARY KEY,
  name VARCHAR(255),
  deleted_at TIMESTAMP NULL
);

SELECT * FROM products WHERE deleted_at IS NULL;  -- Only active

-- SCD Type 2 (slowly changing dimension; track history)
CREATE TABLE customer_dimensions (
  customer_key SERIAL PRIMARY KEY,  -- Surrogate key
  customer_id INT,                  -- Business key
  name VARCHAR(255),
  city VARCHAR(100),
  valid_from DATE,
  valid_to DATE,
  is_current BOOLEAN
);

INSERT INTO customer_dimensions (customer_id, name, city, valid_from, valid_to, is_current)
VALUES (1, 'Alice', 'NYC', '2024-01-01', NULL, true);
-- When Alice moves to LA:
UPDATE customer_dimensions SET valid_to = '2024-06-01', is_current = false 
WHERE customer_id = 1 AND is_current = true;
INSERT INTO customer_dimensions (customer_id, name, city, valid_from, is_current)
VALUES (1, 'Alice', 'LA', '2024-06-02', true);

-- Temporal table (automatic versioning in some DBs; manual in Postgres)
CREATE TABLE documents (
  document_id SERIAL PRIMARY KEY,
  title VARCHAR(255),
  content TEXT,
  version INT DEFAULT 1,
  valid_from TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  valid_to TIMESTAMP
);
```

**Pitfalls:**
- Soft delete without filtering queries = stale data in results.
- Audit table without triggers = manual updates = missed history.
- SCD Type 1 (overwrite) when history needed = no rollback.

---

#### **Topic 19: JSON/JSONB in Postgres**

**Concept**: Store semi-structured data. JSONB (binary JSON) is faster and supports indexing.

**Examples:**
```sql
-- Store metadata
CREATE TABLE products (
  product_id SERIAL PRIMARY KEY,
  metadata JSONB NOT NULL DEFAULT '{}'
);

INSERT INTO products VALUES (1, '{"color": "red", "size": "L", "tags": ["popular", "sale"]}');

-- Access (-> returns JSONB, ->> returns text)
SELECT metadata -> 'color' FROM products;  -- Returns "red" (JSON string)
SELECT metadata ->> 'color' FROM products;  -- Returns red (text)

-- Navigate nested
SELECT metadata -> 'dimensions' ->> 'width' FROM products;

-- Array access
SELECT metadata -> 'tags' -> 0 FROM products;  -- First tag

-- Query (find products with 'sale' tag)
SELECT * FROM products WHERE metadata -> 'tags' @> '["sale"]'::jsonb;

-- JSONPath queries (more powerful)
SELECT * FROM products WHERE jsonb_path_exists(metadata, '$.tags[*] ? (@ == "sale")');

-- Index JSONB for faster queries
CREATE INDEX idx_products_metadata ON products USING GIN(metadata);

-- Aggregate to array
CREATE TABLE orders (
  order_id SERIAL PRIMARY KEY,
  items JSONB ARRAY
);

-- Extract and transform
SELECT jsonb_object_agg(key, value) FROM
  (SELECT key, value FROM jsonb_each(metadata)) t;
```

**Pitfalls:**
- JSON (text) slower than JSONB (binary); use JSONB.
- No indexing support for JSON; only JSONB.
- Overusing JSON = avoiding proper schema design; use structured columns when possible.

---

#### **Topic 20: Stored Procedures & Functions (PL/pgSQL)**

**Concept**: Write logic in the database. Functions are called from queries; procedures are called explicitly.

**Examples:**
```sql
-- Function (returns value)
CREATE OR REPLACE FUNCTION get_book_count(author_id INT)
RETURNS INT AS $$
BEGIN
  RETURN (SELECT COUNT(*) FROM books WHERE books.author_id = $1);
END;
$$ LANGUAGE plpgsql;

SELECT get_book_count(1);

-- Function with OUT parameter
CREATE OR REPLACE FUNCTION get_author_stats(author_id INT, OUT book_count INT, OUT avg_year INT)
AS $$
BEGIN
  SELECT COUNT(*), ROUND(AVG(published_year))
  INTO book_count, avg_year
  FROM books
  WHERE books.author_id = $1;
END;
$$ LANGUAGE plpgsql;

SELECT * FROM get_author_stats(1);

-- Procedure (no return; CALL, not SELECT)
CREATE OR REPLACE PROCEDURE transfer_funds(from_id INT, to_id INT, amount DECIMAL)
LANGUAGE plpgsql
AS $$
BEGIN
  UPDATE accounts SET balance = balance - amount WHERE account_id = from_id;
  UPDATE accounts SET balance = balance + amount WHERE account_id = to_id;
  COMMIT;
EXCEPTION WHEN OTHERS THEN
  ROLLBACK;
  RAISE;
END;
$$;

CALL transfer_funds(1, 2, 100);

-- Trigger function (called by trigger)
CREATE OR REPLACE FUNCTION update_book_count()
RETURNS TRIGGER AS $$
BEGIN
  UPDATE authors SET book_count = (SELECT COUNT(*) FROM books WHERE author_id = NEW.author_id)
  WHERE author_id = NEW.author_id;
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_update_book_count
AFTER INSERT OR DELETE ON books
FOR EACH ROW
EXECUTE FUNCTION update_book_count();
```

**Pitfalls:**
- Logic in DB harder to version control and test; prefer application logic.
- PL/pgSQL errors can be cryptic; test thoroughly.
- Triggers can hide side effects; document them.

---

#### **Topic 21: Query Tuning**

**Concept**: Systematic approach: collect slow queries, isolate the problem, analyze EXPLAIN, fix, and measure.

**Steps:**
1. **Collect**: Enable slow query log or query profiling.
2. **Isolate**: Use EXPLAIN ANALYZE on suspect query.
3. **Diagnose**: Look for Seq Scans, nested loops, high costs.
4. **Fix**: Add indexes, rewrite JOIN, add WHERE, or denormalize.
5. **Measure**: Compare timings before/after; validate correctness.

**Examples:**
```sql
-- Enable slow query logging (postgresql.conf)
log_min_duration_statement = 1000  -- Log queries > 1 second

-- Collect from logs
tail -f /var/log/postgresql/postgresql.log | grep duration

-- Isolate: slow query
SELECT a.name, COUNT(b.book_id) FROM authors a
LEFT JOIN books b ON a.author_id = b.author_id
GROUP BY a.author_id;

-- Analyze
EXPLAIN ANALYZE SELECT a.name, COUNT(b.book_id) FROM authors a
LEFT JOIN books b ON a.author_id = b.author_id
GROUP BY a.author_id;

-- Result: Seq Scan on authors (expensive)
-- Fix: Ensure index on books.author_id
CREATE INDEX idx_books_author ON books(author_id);

-- Re-analyze
EXPLAIN ANALYZE SELECT a.name, COUNT(b.book_id) FROM authors a
LEFT JOIN books b ON a.author_id = b.author_id
GROUP BY a.author_id;

-- Result: Index Scan (faster)
-- Measure: time before/after
\timing
-- Run query, note time
```

**Pitfalls:**
- Tuning based on small dataset = no benefit on large data.
- Adding too many indexes = slows writes.
- Premature optimization = wasted effort; optimize bottlenecks.

---

#### **Topic 22: Security Basics**

**Concept**: Control access via roles and permissions. Use parameterized queries to prevent SQL injection.

**Examples:**
```sql
-- Create roles (users)
CREATE ROLE analyst LOGIN PASSWORD 'secure_password';
CREATE ROLE admin SUPERUSER;

-- Grant permissions
GRANT CONNECT ON DATABASE testdb TO analyst;
GRANT USAGE ON SCHEMA public TO analyst;
GRANT SELECT ON books TO analyst;  -- Analyst can read books
GRANT SELECT, INSERT, UPDATE ON authors TO analyst;  -- Read and write authors

-- Revoke
REVOKE INSERT ON authors FROM analyst;

-- Row-level security (Postgres 9.5+)
CREATE TABLE sensitive_data (
  id SERIAL PRIMARY KEY,
  user_id INT,
  content TEXT
);

ALTER TABLE sensitive_data ENABLE ROW LEVEL SECURITY;

CREATE POLICY user_isolation ON sensitive_data
  FOR SELECT USING (user_id = CURRENT_USER_ID);

-- Now, user 1 only sees their rows
SET ROLE analyst;
SELECT * FROM sensitive_data;  -- Only rows where user_id = analyst's ID

-- SQL injection prevention: use parameterized queries
-- Bad (vulnerable):
SELECT * FROM users WHERE email = 'injected@example.com; DROP TABLE users; --';

-- Good (safe):
-- In application code with parameterized query:
-- conn.execute("SELECT * FROM users WHERE email = ?", [user_email])
```

**Pitfalls:**
- No row-level security = all users see all rows.
- SQL injection from unsanitized input = data breach.
- Too-permissive roles = privilege escalation.

---

### **PART B: NoSQL**

---

#### **Topic 23: CAP Theorem & PACELC**

**Concept**: CAP says you can't have all three: Consistency (all replicas agree), Availability (always respond), Partition-tolerance (survive network splits). PACELC extends: if no Partition, choose Latency vs Consistency; if Partition, choose Availability vs Consistency.

**Examples:**
- **CP (Consistency + Partition)**: PostgreSQL (single master), Zookeeper. Chooses consistency; unavailable during partitions.
- **AP (Availability + Partition)**: Cassandra, DynamoDB, S3. Chooses availability; eventual consistency.
- **CA (Consistency + Availability)**: Impossible in distributed systems.

**Use NoSQL when:**
- Horizontal scaling needed (more machines, not bigger machine).
- Semi-structured or unstructured data (documents, events).
- Extreme write throughput (IoT sensors, logs).
- Low-latency reads (in-memory caches).

**Pitfalls:**
- Assuming NoSQL = no transactions (many now support them).
- Ignoring consistency models = silent data loss.

---

#### **Topic 24: Redis**

**Concept**: In-memory key-value store. Fast (microseconds), supports rich data types (strings, hashes, lists, sets, sorted sets, streams), persistence, pub/sub.

**Examples:**
```bash
# Install and run
docker run --name redis -p 6379:6379 -d redis:7

# Connect
redis-cli

# Strings
SET key1 "hello"
GET key1
INCR counter
APPEND key1 " world"

# Hashes (maps)
HSET user:1 name "Alice" age 30 email "alice@example.com"
HGET user:1 name
HGETALL user:1

# Lists
LPUSH tasks "task1" "task2"
RPUSH tasks "task3"
LRANGE tasks 0 -1
LPOP tasks

# Sets (unique members, no order)
SADD tags "python" "database" "nosql"
SMEMBERS tags
SISMEMBER tags "python"
SINTER tags tags2  -- Intersection

# Sorted sets (scored members)
ZADD leaderboard 100 "alice" 200 "bob" 150 "charlie"
ZRANGE leaderboard 0 -1 WITHSCORES
ZRANK leaderboard "alice"

# Streams (event log)
XADD events "*" sensor "temp" value "23.5"
XADD events "*" sensor "humidity" value "60"
XRANGE events - +  -- All events
XREAD STREAMS events 0  -- Read all

# Pub/Sub
PUBLISH channel:updates "New message"
SUBSCRIBE channel:updates  -- Receive

# Expiration (TTL)
SETEX session:user1 3600 "{...json...}"  -- Expire in 1 hour
TTL session:user1

# Transactions
MULTI
SET key1 value1
SET key2 value2
EXEC

# Persistence
SAVE  -- Synchronous save
BGSAVE  -- Async save
```

**Python example:**
```python
import redis
r = redis.Redis(host='localhost', port=6379)
r.set('key1', 'hello')
print(r.get('key1'))  # b'hello'

# List
r.lpush('tasks', 'task1', 'task2')
print(r.lrange('tasks', 0, -1))  # [b'task2', b'task1']

# Publish/Subscribe
pub = r.pubsub()
pub.subscribe('channel')
for msg in pub.listen():
    print(msg)
```

**Pitfalls:**
- No persistence by default = data lost on restart; enable RDB or AOF.
- Single-threaded = all operations sequential (but atomic).
- RAM bound = can't store massive datasets (use Valkey/cluster for scaling).
- Pub/Sub is fire-and-forget (no history); use Streams for durability.

---

#### **Topic 25: MongoDB**

**Concept**: Document store; JSON-like documents (BSON). Flexible schema, fast writes, good for hierarchical data.

**Examples:**
```bash
# Install and run
docker run --name mongo -p 27017:27017 -d mongo:6

# Connect via MongoDB shell
mongosh

# Databases and collections
use mydb
db.createCollection('users')

# Insert
db.users.insertOne({ name: 'Alice', age: 30, email: 'alice@example.com' })
db.users.insertMany([
  { name: 'Bob', age: 25 },
  { name: 'Charlie', age: 35 }
])

# Find (Query)
db.users.findOne({ name: 'Alice' })
db.users.find({ age: { $gte: 30 } })  # age >= 30
db.users.find({ name: { $in: ['Alice', 'Bob'] } })

# Update
db.users.updateOne({ name: 'Alice' }, { $set: { age: 31 } })
db.users.updateMany({ age: { $lt: 30 } }, { $set: { status: 'junior' } })

# Delete
db.users.deleteOne({ name: 'Charlie' })
db.users.deleteMany({ status: 'inactive' })

# Aggregation pipeline (powerful data transformation)
db.orders.aggregate([
  { $match: { status: 'completed' } },
  { $group: { _id: '$customer_id', total: { $sum: '$amount' }, count: { $sum: 1 } } },
  { $sort: { total: -1 } },
  { $limit: 10 }
])

# Another pipeline: denormalize products into orders
db.orders.aggregate([
  { $lookup: {
      from: 'products',
      localField: 'product_id',
      foreignField: '_id',
      as: 'product_details'
    }
  },
  { $unwind: '$product_details' }
])

# Indexes
db.users.createIndex({ email: 1 })  -- Unique
db.users.createIndex({ age: 1, name: 1 })  -- Compound
db.users.createIndex({ 'address.city': 1 })  -- Nested field

# Transactions (multi-doc, Mongo 4.0+)
session = db.getMongo().startSession()
session.startTransaction()
db.users.insertOne({ name: 'David' }, { session })
db.orders.insertOne({ user_id: 'David_id', amount: 100 }, { session })
session.commitTransaction()
```

**Python example:**
```python
from pymongo import MongoClient

client = MongoClient('mongodb://localhost:27017/')
db = client['mydb']
users = db['users']

# Insert
users.insert_one({'name': 'Alice', 'age': 30})

# Find
for user in users.find({'age': {'$gte': 30}}):
    print(user)

# Aggregation
pipeline = [
    {'$match': {'age': {'$gte': 30}}},
    {'$group': {'_id': None, 'avg_age': {'$avg': '$age'}}}
]
result = list(users.aggregate(pipeline))
print(result)
```

**Schema design:**
```
-- Embed (one-to-few relationship)
{
  _id: 1,
  name: 'Alice',
  addresses: [
    { street: '123 Main', city: 'NYC' },
    { street: '456 Oak', city: 'LA' }
  ]
}

-- Reference (one-to-many relationship; avoid N+1)
User: { _id: 1, name: 'Alice' }
Address: { _id: 100, user_id: 1, street: '123 Main' }
Address: { _id: 101, user_id: 1, street: '456 Oak' }
```

**Pitfalls:**
- No ACID transactions pre-4.0; use application-level consistency.
- Embedding large docs = slow queries and memory bloat; reference instead.
- No joins in queries pre-$lookup; must denormalize or make application joins.
- Indexing missing = full collection scan; create indexes proactively.

---

#### **Topic 26: Cassandra & DynamoDB — Wide-Column Stores**

**Concept**: Distributed, partition-first design. Query is determined by the data model (query-first design). Scale horizontally with no single point of failure.

**Cassandra example:**
```bash
# Run Cassandra
docker run --name cassandra -p 9042:9042 -d cassandra:4

# Connect
cqlsh localhost

# Create keyspace (DB)
CREATE KEYSPACE ecommerce WITH replication = {'class': 'SimpleStrategy', 'replication_factor': 3};
USE ecommerce;

-- Partition key (how data is distributed) + clustering key (sort within partition)
CREATE TABLE products_by_category (
  category TEXT,               -- Partition key
  product_id UUID,             -- Clustering key
  name TEXT,
  price DECIMAL,
  PRIMARY KEY (category, product_id)
);

-- Query-first design: table is optimized for "get products in category"
SELECT * FROM products_by_category WHERE category = 'electronics';

-- To query by price, create a different table
CREATE TABLE products_by_price (
  price DECIMAL,
  product_id UUID,
  category TEXT,
  name TEXT,
  PRIMARY KEY (price, product_id)
);

-- Insert
INSERT INTO products_by_category (category, product_id, name, price)
VALUES ('electronics', uuid(), 'Laptop', 1200.00);

-- Query
SELECT * FROM products_by_category WHERE category = 'electronics' LIMIT 10;
```

**DynamoDB (AWS) example:**
```
-- Same concept: Partition Key + Sort Key
Table: Orders
PK: CustomerID (partition), OrderDate (sort)

Query by customer and date:
query(
  KeyConditionExpression='CustomerID = :cid AND OrderDate > :date',
  ExpressionAttributeValues={
    ':cid': 'cust123',
    ':date': '2024-01-01'
  }
)

-- GSI (Global Secondary Index) for alternative queries
GSI: OrderStatus-OrderDate
PK: OrderStatus, SK: OrderDate

Query by status:
query(
  IndexName='OrderStatus-OrderDate',
  KeyConditionExpression='OrderStatus = :status',
  ExpressionAttributeValues={':status': 'pending'}
)

-- Avoid hotspots: don't use sequential PK
Bad:  PK = timestamp (all writes to same partition)
Good: PK = timestamp + random suffix
```

**Pitfalls:**
- Querying by non-key column = full table scan (slow).
- Hotspot: all writes to one partition = bottleneck.
- Over-provisioning throughput = expensive; use on-demand for variable loads.

---

#### **Topic 27: DynamoDB Specifics**

**Concept**: AWS's fully managed key-value store. Single-table design is an advanced pattern.

**Examples:**
```
-- Single-table design: one table, multiple entity types
Table: Application
PK: EntityType#EntityID
SK: Timestamp#SortAttribute

Item 1: { PK: 'USER#user123', SK: 'PROFILE#2024-01-01', name: 'Alice', email: '...' }
Item 2: { PK: 'USER#user123', SK: 'ORDER#order456', date: '2024-03-15', total: 100 }
Item 3: { PK: 'ORDER#order456', SK: 'ITEM#item789', product: 'Laptop', qty: 1 }

-- Query patterns:
1. Get user profile: query(PK='USER#user123', SK begins_with 'PROFILE')
2. Get user orders: query(PK='USER#user123', SK begins_with 'ORDER')
3. Get order items: query(PK='ORDER#order456', SK begins_with 'ITEM')

-- LSI (Local Secondary Index): alternate sort key for same partition
Table: Orders
PK: CustomerID, SK: OrderDate
LSI: CustomerID, SK: TotalAmount  -- Query by customer + amount

-- GSI (Global Secondary Index): alternate PK + SK
GSI: OrderStatus, OrderDate  -- Query by status, sorted by date

-- Capacity modes:
1. Provisioned: predict peak, pay per unit; cheaper if predictable
2. On-Demand: pay per request; pricier but flexible

-- TTL (Time to Live): auto-delete old items
Attribute: ExpirationTime (unix timestamp)
DynamoDB scans and deletes expired items (eventual; not immediate)
```

**Pitfalls:**
- Single-table design adds query complexity; only use if multiple access patterns.
- Exceeding provisioned throughput = throttled (slow requests).
- On-Demand mode = no cold starts, but pricier at scale.

---

#### **Topic 28: Neo4j — Graph Database**

**Concept**: Nodes and relationships. Powerful for hierarchical data (org charts, recommendations).

**Examples:**
```
-- Cypher query language

-- Create nodes
CREATE (alice:Person { name: 'Alice', age: 30 })
CREATE (bob:Person { name: 'Bob', age: 25 })
CREATE (companyA:Company { name: 'TechCorp' })

-- Create relationships
MATCH (alice:Person { name: 'Alice' })
MATCH (companyA:Company { name: 'TechCorp' })
CREATE (alice)-[:WORKS_AT { since: 2020 }]->(companyA)

-- Query: find all people working at TechCorp
MATCH (p:Person)-[:WORKS_AT]->(c:Company { name: 'TechCorp' })
RETURN p.name, p.age

-- Find friends of Alice (2 hops)
MATCH (alice:Person { name: 'Alice' })-[:KNOWS]-()-[:KNOWS]-(friend)
RETURN DISTINCT friend.name

-- Shortest path
MATCH p = shortestPath((alice:Person)-[*]-(bob:Person))
RETURN p

-- Aggregation
MATCH (p:Person)-[:WORKS_AT]->(c:Company)
RETURN c.name, COUNT(p) as num_employees
ORDER BY num_employees DESC
```

**Pitfalls:**
- Unbounded traversals (no LIMIT) = expensive queries.
- Not all relationships are appropriate for graphs; RDBMS better for tabular data.

---

#### **Topic 29: Time-Series DBs (InfluxDB, Timescale)**

**Concept**: Optimized for time-stamped data. High throughput, good for compression.

**Examples:**
```
-- InfluxDB (line protocol)
measurement,tag1=value1,tag2=value2 field1=10,field2=20 timestamp

-- Example
temperature,location=NYC,sensor=1 value=23.5 1609459200000000000

-- Query (InfluxQL or Flux)
SELECT value FROM temperature WHERE location='NYC' AND time > '2024-01-01'

-- Timescale (PostgreSQL extension)
CREATE TABLE metrics (
  time TIMESTAMPTZ NOT NULL,
  device_id TEXT NOT NULL,
  temperature FLOAT,
  PRIMARY KEY (time, device_id)
);

SELECT create_hypertable('metrics', 'time');  -- Convert to hypertable

INSERT INTO metrics VALUES ('2024-01-01 10:00:00', 'device1', 23.5);

-- Query with time_bucket
SELECT time_bucket('1 hour', time) as hour, device_id, AVG(temperature)
FROM metrics
GROUP BY hour, device_id
ORDER BY hour DESC;

-- Compression (auto)
ALTER TABLE metrics SET (timescaledb.compress = true);
SELECT compress_chunk(show_chunks('metrics'));
```

**Pitfalls:**
- Time-series DBs poor for non-time queries; use alongside OLTP DB.
- Unbounded retention = bloat; set TTL/downsampling policies.

---

#### **Topic 30: Elasticsearch / OpenSearch**

**Concept**: Inverted index for full-text search. Fast keyword searches, good for logs and analytics.

**Examples:**
```bash
# Run Elasticsearch
docker run --name elasticsearch -p 9200:9200 -e "discovery.type=single-node" -d docker.elastic.co/elasticsearch/elasticsearch:8.0.0

# Create index with mapping
curl -X PUT "localhost:9200/articles" -H 'Content-Type: application/json' -d '{
  "mappings": {
    "properties": {
      "title": { "type": "text" },
      "content": { "type": "text", "analyzer": "standard" },
      "tags": { "type": "keyword" },
      "published_date": { "type": "date" }
    }
  }
}'

# Index document
curl -X POST "localhost:9200/articles/_doc" -H 'Content-Type: application/json' -d '{
  "title": "Intro to Elasticsearch",
  "content": "Elasticsearch is a search engine...",
  "tags": ["search", "database"],
  "published_date": "2024-01-15"
}'

# Search: keyword match
curl -X GET "localhost:9200/articles/_search" -H 'Content-Type: application/json' -d '{
  "query": {
    "match": {
      "content": "search engine"
    }
  }
}'

# Search: bool query (AND/OR)
curl -X GET "localhost:9200/articles/_search" -H 'Content-Type: application/json' -d '{
  "query": {
    "bool": {
      "must": [{"match": {"content": "elasticsearch"}}],
      "filter": [{"term": {"tags": "database"}}]
    }
  }
}'

# Aggregation (bucketing)
curl -X GET "localhost:9200/articles/_search" -H 'Content-Type: application/json' -d '{
  "aggs": {
    "by_tag": {
      "terms": {"field": "tags"}
    }
  }
}'
```

**Pitfalls:**
- Inverted index good for text; bad for numeric range queries (use Lucene range queries instead).
- Sharding strategy important; bad sharding = slow queries.

---

#### **Topic 31: Vector Databases**

**Concept**: Store and search dense vectors (embeddings). Used for semantic search, recommendations.

**pgvector example:**
```sql
-- Install pgvector extension
CREATE EXTENSION vector;

-- Create table with vector column
CREATE TABLE documents (
  doc_id SERIAL PRIMARY KEY,
  title TEXT,
  embedding vector(1536)  -- OpenAI embedding dimension
);

-- Insert vectors (simulate embeddings from OpenAI)
INSERT INTO documents (title, embedding) VALUES
('Intro to ML', '[0.1, 0.2, 0.3, ...]'::vector),
('Advanced SQL', '[0.2, 0.3, 0.4, ...]'::vector);

-- Cosine distance index (HNSW)
CREATE INDEX ON documents USING hnsw (embedding vector_cosine_ops);

-- Search: find similar documents
SELECT doc_id, title, 1 - (embedding <=> '[0.15, 0.25, 0.35, ...]'::vector) as similarity
FROM documents
ORDER BY embedding <=> '[0.15, 0.25, 0.35, ...]'::vector
LIMIT 5;
```

**Python with Pinecone:**
```python
from pinecone import Pinecone
import openai

pc = Pinecone(api_key='...')
index = pc.Index('documents')

# Embed text
query_text = 'machine learning tutorial'
embedding = openai.Embedding.create(input=query_text, model='text-embedding-ada-002')['data'][0]['embedding']

# Search
results = index.query(vector=embedding, top_k=5)
for match in results['matches']:
    print(f"Doc {match['id']}: score {match['score']}")
```

**Pitfalls:**
- Vector similarity is not transitive (A similar to B, B similar to C, doesn't mean A similar to C).
- Embedding quality matters; use established models (OpenAI, Huggingface).

---

#### **Topic 32: Polyglot Persistence & Choosing the Right DB**

**Concept**: Use multiple databases for different workloads. RDBMS for transactional consistency; cache for speed; document store for flexibility; search engine for text queries.

**Common stack:**
```
PostgreSQL        -> Transactional data (orders, users, inventory)
Redis             -> Cache (session, leaderboard) + rate limiting + pub/sub
MongoDB           -> Activity logs, user events, audit trails
Elasticsearch     -> Full-text search (articles, logs)
S3                -> Files, backups
DynamoDB          -> High-throughput write scenarios
pgvector          -> Semantic search
```

**Decision tree:**
```
Is the data transactional (ACID)?
  YES -> PostgreSQL, MySQL
  NO  -> Is the data unstructured (documents)?
           YES -> MongoDB, DynamoDB
           NO  -> Is the data hierarchical (graphs)?
                    YES -> Neo4j
                    NO  -> Is the data time-series?
                             YES -> InfluxDB, Timescale
                             NO  -> Is it text search?
                                      YES -> Elasticsearch
                                      NO  -> Is it a cache?
                                               YES -> Redis
                                               NO  -> Default to RDBMS
```

**Pitfalls:**
- Too many databases = operational complexity (monitoring, backups, learning curve).
- Splitting data across DBs without clear boundaries = distributed transactions = pain.
- Eventual consistency across DBs without a sync strategy = stale data.

---

### **PART C: DATABASE TESTING & QA**

---

#### **Topic 33: What to Test in a Database Layer**

**Concept**: Schema, constraints, data quality, migrations, performance, replication.

**Checklist:**
- Schema: tables exist, columns have correct types and defaults.
- Constraints: primary keys, foreign keys, unique constraints, not null, checks.
- Data quality: no orphaned rows, valid ranges, no duplicates where unexpected.
- Migrations: forward and backward work; no data loss; zero-downtime deploys possible.
- Performance: queries execute within SLA; indexes effective; no N+1 queries.
- Replication: replicas stay in sync; failover works; no data loss.
- Security: no unauthorized access; PII encrypted; audit trails.

---

#### **Topic 34: Unit Testing DB Code**

**Concept**: Test DB logic in isolation using in-memory DBs or Testcontainers.

**In-Memory H2 (Java example):**
```java
@DataSource(url = "jdbc:h2:mem:test", driver = "org.h2.Driver")
public class AuthorRepositoryTest {
  @Before
  public void setUp() {
    // Create schema
    db.execute("CREATE TABLE authors (author_id INT PRIMARY KEY, name VARCHAR(255))");
  }

  @Test
  public void testInsertAuthor() {
    authorRepo.insert(new Author(1, "Jane Austen"));
    Author result = authorRepo.findById(1);
    assertEquals("Jane Austen", result.getName());
  }
}
```

**Testcontainers (Java, real Postgres):**
```java
@Testcontainers
public class AuthorRepositoryTest {
  @Container
  static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15")
    .withDatabaseName("testdb")
    .withUsername("test")
    .withPassword("test");

  private DataSource dataSource;
  private AuthorRepository authorRepo;

  @Before
  public void setUp() throws Exception {
    dataSource = new HikariDataSource(...postgres.getJdbcUrl()...);
    authorRepo = new AuthorRepository(dataSource);
    // Run migrations
    Flyway.configure().dataSource(dataSource).load().migrate();
  }

  @Test
  public void testInsertAuthor() {
    authorRepo.insert(new Author(1, "Jane Austen"));
    Author result = authorRepo.findById(1);
    assertEquals("Jane Austen", result.getName());
  }

  @After
  public void tearDown() {
    dataSource.close();
  }
}
```

**Python with pytest & Docker:**
```python
import pytest
import psycopg2
from testcontainers.postgres import PostgresContainer

@pytest.fixture
def postgres_db():
    with PostgresContainer("postgres:15") as postgres:
        conn = psycopg2.connect(postgres.get_connection_url())
        cursor = conn.cursor()
        cursor.execute("CREATE TABLE authors (author_id SERIAL PRIMARY KEY, name VARCHAR(255))")
        conn.commit()
        yield conn
        conn.close()

def test_insert_author(postgres_db):
    cursor = postgres_db.cursor()
    cursor.execute("INSERT INTO authors (name) VALUES (%s) RETURNING author_id", ("Jane Austen",))
    author_id = cursor.fetchone()[0]
    postgres_db.commit()
    
    cursor.execute("SELECT name FROM authors WHERE author_id = %s", (author_id,))
    result = cursor.fetchone()[0]
    assert result == "Jane Austen"
```

**Pitfalls:**
- Testing with different DB in prod (in-memory vs real) = surprises in production.
- Not cleaning up data between tests = test interdependencies.
- Slow tests = developers skip them; use transactions + rollback for speed.

---

#### **Topic 35: Migrations (Flyway, Liquibase)**

**Concept**: Version-control schema changes. Up migrations add changes; down migrations revert.

**Flyway example:**
```
# Folder structure
migrations/
  V1__Create_authors_table.sql
  V2__Create_books_table.sql
  V3__Add_isbn_to_books.sql
  U3__Drop_isbn_column.sql  (undo)

# Flyway tracks applied migrations in flyway_schema_history table
```

**V1__Create_authors_table.sql:**
```sql
CREATE TABLE authors (
  author_id SERIAL PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  birth_date DATE
);
```

**V2__Create_books_table.sql:**
```sql
CREATE TABLE books (
  book_id SERIAL PRIMARY KEY,
  title VARCHAR(500) NOT NULL,
  author_id INT NOT NULL REFERENCES authors(author_id),
  published_year INT
);
```

**Java config:**
```java
Flyway flyway = Flyway.configure()
  .dataSource(dataSource)
  .locations("db/migration")
  .load();

int migrationsApplied = flyway.migrate();
System.out.println(migrationsApplied + " migrations applied");
```

**Zero-downtime migration (adding NOT NULL column):**
```sql
-- v1: add column with default
ALTER TABLE products ADD COLUMN new_field VARCHAR(255) DEFAULT 'pending';

-- v2: backfill in batches (application continues serving)
UPDATE products SET new_field = compute_value(id) WHERE new_field = 'pending' LIMIT 1000;

-- v3: make not null
ALTER TABLE products ALTER COLUMN new_field SET NOT NULL;
```

**Pitfalls:**
- Writing down migrations without testing them = can't rollback.
- Mixing schema + data migrations in one file = hard to rollback partially.
- Flyway/Liquibase doesn't handle all databases equally; test against target DB.

---

#### **Topic 36: Seed & Teardown Strategies**

**Concept**: Set up test data before tests; clean up after.

**Transactional rollback (fastest):**
```python
@pytest.fixture
def db_transaction():
    conn = psycopg2.connect(...)
    conn.autocommit = False
    cursor = conn.cursor()
    
    # Setup
    cursor.execute("INSERT INTO authors (name) VALUES ('Jane Austen')")
    conn.commit()
    
    yield cursor
    
    # Teardown: rollback to before test
    conn.rollback()
    conn.close()

def test_with_author(db_transaction):
    db_transaction.execute("SELECT COUNT(*) FROM authors")
    assert db_transaction.fetchone()[0] == 1
```

**Seed file approach:**
```sql
-- seed.sql
TRUNCATE TABLE books CASCADE;
TRUNCATE TABLE authors CASCADE;

INSERT INTO authors (author_id, name) VALUES
  (1, 'Jane Austen'),
  (2, 'Mark Twain');

INSERT INTO books (book_id, title, author_id, published_year) VALUES
  (1, 'Pride and Prejudice', 1, 1813),
  (2, 'Adventures of Tom Sawyer', 2, 1876);
```

**Load before test:**
```python
@pytest.fixture
def populated_db():
    conn = psycopg2.connect(...)
    with open('seed.sql') as f:
        conn.cursor().execute(f.read())
    conn.commit()
    yield conn
    conn.close()
```

**Pitfalls:**
- Not cleaning up = tests interfere with each other.
- Seed data not representative = tests miss real-world issues.
- Seeding takes longer than test execution = slow suite.

---

#### **Topic 37: Schema Contract Tests**

**Concept**: Assert schema structure: tables exist, columns have correct types, nullable/not nullable, defaults.

**SQL example:**
```sql
-- Assert authors table exists
SELECT table_name FROM information_schema.tables 
WHERE table_schema='public' AND table_name='authors'
LIMIT 1;

-- Assert author_id is INT NOT NULL PRIMARY KEY
SELECT column_name, data_type, is_nullable, column_default
FROM information_schema.columns
WHERE table_name='authors' AND column_name='author_id';

-- Assert email is UNIQUE
SELECT constraint_name FROM information_schema.table_constraints
WHERE table_name='authors' AND constraint_type='UNIQUE';
```

**Python with pytest:**
```python
import psycopg2

@pytest.fixture
def schema_db():
    return psycopg2.connect(...)

def test_authors_table_exists(schema_db):
    cursor = schema_db.cursor()
    cursor.execute("""
        SELECT table_name FROM information_schema.tables 
        WHERE table_schema='public' AND table_name='authors'
    """)
    assert cursor.fetchone() is not None

def test_author_id_is_primary_key(schema_db):
    cursor = schema_db.cursor()
    cursor.execute("""
        SELECT constraint_type FROM information_schema.table_constraints 
        WHERE table_name='authors' AND constraint_name='authors_pkey'
    """)
    assert cursor.fetchone()[0] == 'PRIMARY KEY'

def test_email_column_unique(schema_db):
    cursor = schema_db.cursor()
    cursor.execute("""
        SELECT constraint_name FROM information_schema.table_constraints 
        WHERE table_name='authors' AND constraint_type='UNIQUE' 
        AND constraint_name LIKE '%email%'
    """)
    assert cursor.fetchone() is not None
```

**Pitfalls:**
- Contract tests strict; breaks on intentional schema changes.
- Not testing enough constraints = false confidence.

---

#### **Topic 38: Data Quality Testing (Great Expectations, dbt, Soda)**

**Concept**: Validate data after transformation (no nulls where unexpected, values in range, uniqueness).

**Great Expectations example:**
```python
import great_expectations as ge

# Load data
df = ge.read_csv('data.csv')

# Define expectations
df.expect_column_values_to_be_in_set('status', ['active', 'inactive'])
df.expect_column_values_to_be_between('age', min_value=0, max_value=120)
df.expect_column_values_to_be_not_null('email')
df.expect_column_values_to_match_regex('email', regex=r'^[^@]+@[^@]+\.[^@]+$')

# Validate
validation_result = df.validate()
print(validation_result)
```

**dbt tests (YAML in dbt project):**
```yaml
# models/schema.yml
models:
  - name: users
    columns:
      - name: user_id
        tests:
          - unique
          - not_null
      - name: email
        tests:
          - unique
          - not_null
      - name: age
        tests:
          - accepted_values:
              values: [18, 21, 25, 30]  # or custom
```

**Soda SQL:**
```sql
checks for orders:
  - missing_count(order_date) = 0
  - duplicate_count(order_id) = 0
  - invalid_count(status) where status NOT IN ('pending', 'completed', 'canceled') = 0
  - avg(total_amount) > 10
```

**Pitfalls:**
- Data quality rules without domain knowledge = false positives.
- Not running tests in CI/CD = bugs slip to production.

---

#### **Topic 39: Performance Testing**

**Concept**: Verify queries meet SLA. Use pgbench for load testing; detect N+1 queries; analyze slow-query logs.

**pgbench example:**
```bash
# Initialize benchmark database
pgbench -i -s 10 testdb  # -s 10 = scale (10 * 100k rows)

# Run benchmark (100 clients, 1000 transactions per client)
pgbench -c 100 -t 1000 testdb

# Custom script
cat > custom.sql << 'EOF'
SELECT * FROM accounts WHERE id = :id;
UPDATE accounts SET balance = balance - 100 WHERE id = :id;
EOF

pgbench -f custom.sql -c 50 -t 500 testdb
```

**Detect N+1 queries (ORM issue):**
```python
# Bad: 1 query to get authors, then N queries for books
authors = db.query(Author).all()
for author in authors:
    books = db.query(Book).filter(Book.author_id == author.id).all()  # N+1!

# Good: use JOIN or eager loading
authors = db.query(Author).options(joinedload(Author.books)).all()
```

**Slow-query log:**
```sql
-- Enable in postgresql.conf
log_min_duration_statement = 1000  -- Log queries > 1 second
log_statement = 'all'              -- Log all statements

-- In Postgres logs
tail -f /var/log/postgresql/postgresql.log | grep duration

-- Or use auto_explain
LOAD 'auto_explain';
SET auto_explain.log_min_duration = 1000;
```

**Pitfalls:**
- Load testing with unrealistic data = doesn't reflect production.
- Not testing peak scenarios = surprises in production.

---

#### **Topic 40: Replication & Backup**

**Concept**: What QA should verify.

**Replication:**
```bash
# PostgreSQL streaming replication
# On primary:
wal_level = replica
max_wal_senders = 3
wal_keep_segments = 64

# On standby (recovery.conf):
standby_mode = 'on'
primary_conninfo = 'host=primary_ip port=5432 user=replication password=...'
recovery_target_timeline = 'latest'

# Verify replication lag
SELECT now() - pg_last_wal_receive_lsn() as replication_lag;

# Test failover: promote standby if primary down
pg_ctl promote -D /var/lib/postgresql/data
```

**QA checks:**
- Standby data matches primary (via sampling queries).
- Replication lag < acceptable threshold (e.g., < 10 seconds).
- Failover succeeds; no data loss.

**Backup/restore:**
```bash
# Full backup
pg_dump testdb > backup.sql
pg_dump -Fc testdb > backup.dump  # Custom format, more compact

# Restore
psql testdb < backup.sql
pg_restore -d testdb backup.dump

# Verify: row count should match
SELECT COUNT(*) FROM authors;  -- Should match pre-backup count
```

**Pitfalls:**
- Backup taken but never restored = will fail when needed.
- Backup takes exclusive lock = unavailable during backup.
- Restore not tested = data loss if disaster strikes.

---

#### **Topic 41: PII & GDPR in Test Data**

**Concept**: Anonymize or synthesize test data. Never use production PII in development.

**Anonymization (PostgreSQL):**
```sql
-- Mask email
UPDATE users SET email = CONCAT(uuid_generate_v4(), '@example.com');

-- Truncate phone
UPDATE users SET phone = SUBSTRING(phone, 1, 3) || '-****';

-- Shift dates (lose specificity, keep relative distances)
UPDATE users SET birth_date = birth_date - INTERVAL '10 years';
```

**Synthesize data (Faker):**
```python
from faker import Faker

fake = Faker()

def seed_users(num=1000):
    conn = psycopg2.connect(...)
    cursor = conn.cursor()
    for _ in range(num):
        cursor.execute(
            "INSERT INTO users (name, email, phone) VALUES (%s, %s, %s)",
            (fake.name(), fake.email(), fake.phone_number())
        )
    conn.commit()
    conn.close()
```

**Pitfalls:**
- Pseudo-anonymized data that can be re-identified = not GDPR compliant.
- Real PII in test data = compliance violation.

---

#### **Topic 42: NoSQL Testing Patterns**

**Concept**: Similar to SQL, but adapted for document and wide-column stores.

**MongoDB with Testcontainers (Python):**
```python
import pytest
from testcontainers.mongodb import MongoDbContainer

@pytest.fixture
def mongo_db():
    with MongoDbContainer("mongo:6") as mongo:
        client = MongoClient(mongo.get_connection_url())
        db = client['testdb']
        yield db
        client.close()

def test_insert_user(mongo_db):
    users = mongo_db['users']
    users.insert_one({'name': 'Alice', 'age': 30})
    result = users.find_one({'name': 'Alice'})
    assert result['age'] == 30

def test_aggregation(mongo_db):
    users = mongo_db['users']
    users.insert_many([
        {'name': 'Alice', 'age': 30},
        {'name': 'Bob', 'age': 25}
    ])
    result = list(users.aggregate([
        {'$group': {'_id': None, 'avg_age': {'$avg': '$age'}}}
    ]))
    assert result[0]['avg_age'] == 27.5
```

**DynamoDB Local (for testing):**
```bash
# Run DynamoDB Local
docker run --name dynamodb -p 8000:8000 amazon/dynamodb-local:latest

# Use in tests (Python)
import boto3

@pytest.fixture
def dynamodb():
    dynamodb = boto3.resource(
        'dynamodb',
        endpoint_url='http://localhost:8000',
        region_name='us-east-1',
        aws_access_key_id='testing',
        aws_secret_access_key='testing'
    )
    yield dynamodb

def test_put_item(dynamodb):
    table = dynamodb.Table('orders')
    table.put_item(Item={'CustomerID': 'cust1', 'OrderDate': '2024-01-01'})
    response = table.get_item(Key={'CustomerID': 'cust1', 'OrderDate': '2024-01-01'})
    assert response['Item']['CustomerID'] == 'cust1'
```

**Redis Testcontainers (Java):**
```java
@Testcontainers
public class RedisTest {
  @Container
  static GenericContainer redis = new GenericContainer<>(DockerImageName.parse("redis:7"))
    .withExposedPorts(6379);

  @Test
  public void testSetAndGet() {
    Jedis jedis = new Jedis(redis.getHost(), redis.getFirstMappedPort());
    jedis.set("key", "value");
    assertEquals("value", jedis.get("key"));
    jedis.close();
  }
}
```

**Pitfalls:**
- NoSQL tests often skip validation; schema-less != schema-free.
- Not testing aggregations; they're complex and bug-prone.

---

#### **Topic 43: Common Interview Questions & Gotchas**

**Q: Difference between SQL and NoSQL?**
A: SQL (relational) enforces schema, supports ACID, scales vertically; NoSQL (document/key-value) has flexible schema, eventual consistency, scales horizontally. Choice depends on workload.

**Q: How do you optimize a slow query?**
A: EXPLAIN ANALYZE, identify sequential scans, add indexes on filter/join columns, refactor JOINs, denormalize if needed. Measure before and after.

**Q: What are isolation levels?**
A: READ UNCOMMITTED (dirty reads), READ COMMITTED (no dirty reads), REPEATABLE READ (no phantoms, Postgres), SERIALIZABLE (fully isolated). Higher = slower.

**Q: Explain CAP theorem.**
A: Consistency (all replicas agree), Availability (always respond), Partition-tolerance (survive network splits). Can't have all three; trade-offs vary by system.

**Q: When would you use denormalization?**
A: When read performance critical and write overhead acceptable. Example: store computed aggregates (book_count on authors table) to avoid expensive GROUP BY.

**Q: Difference between INNER JOIN and LEFT JOIN?**
A: INNER = only matching rows; LEFT = all rows from left table (nulls for unmatched right).

**Q: How do you prevent N+1 queries?**
A: Use JOINs, batch queries, or eager loading (ORM joinedload/prefetch_related).

**Q: What's a covering index?**
A: Index that includes non-key columns, allowing query to be answered from index alone (no table lookup). Example: `CREATE INDEX idx ON books(author_id) INCLUDE (title, year)`.

**Q: Difference between NoSQL and RDBMS?**
A: RDBMS: structured, ACID, relational; NoSQL: flexible, eventual consistency, horizontal scale. Use RDBMS for transactional systems; NoSQL for massive scale or unstructured data.

**Q: How do you handle concurrent updates?**
A: Optimistic locking (version column, check before update) or pessimistic locking (SELECT FOR UPDATE). Use transactions to ensure atomicity.

---

## 4. Exercises

### **Beginner**

**Exercise 4.1: Postgres Setup & Sample Queries**

(a) Install Postgres via Docker and create a `library` database.

(b) Create a `library.sql` script:
```sql
CREATE TABLE authors (
  author_id SERIAL PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  birth_year INT
);

CREATE TABLE books (
  book_id SERIAL PRIMARY KEY,
  title VARCHAR(500) NOT NULL,
  author_id INT NOT NULL REFERENCES authors(author_id),
  published_year INT CHECK (published_year > 0)
);

INSERT INTO authors VALUES (1, 'Jane Austen', 1775), (2, 'Mark Twain', 1835);
INSERT INTO books VALUES 
  (1, 'Pride and Prejudice', 1, 1813),
  (2, 'Sense and Sensibility', 1, 1811),
  (3, 'Adventures of Tom Sawyer', 2, 1876);
```

(c) Write 20 SELECT queries:
- Select all books.
- Select books by a specific author.
- Select books published after 1850.
- Use LIKE on title.
- Use ORDER BY, LIMIT, OFFSET.
- Use COUNT, SUM, AVG, MIN, MAX.
- Select distinct authors.
- Select with CASE WHEN.
- etc.

**Exercise 4.2: Schema Modeling**

Design a blog schema with:
- `users` (user_id, username, email, created_at)
- `posts` (post_id, user_id, title, content, created_at)
- `comments` (comment_id, post_id, user_id, text, created_at)

Define primary keys, foreign keys, and constraints. Test it with sample inserts and queries.

---

### **Intermediate**

**Exercise 4.3: Window Functions**

Write queries to:
- Rank authors by number of books using ROW_NUMBER(), RANK(), DENSE_RANK().
- Calculate running total of books by publication year.
- Get top 2 books per author by publication year.
- Use LAG/LEAD to compare adjacent years' book counts.

**Exercise 4.4: CTEs & Aggregations**

Write a CTE-driven reporting query:
```sql
WITH author_stats AS (
  SELECT author_id, COUNT(*) as book_count, AVG(published_year) as avg_year
  FROM books GROUP BY author_id
),
prolific_authors AS (
  SELECT author_id, book_count FROM author_stats WHERE book_count > 2
)
SELECT a.name, pa.book_count
FROM authors a
JOIN prolific_authors pa ON a.author_id = pa.author_id
ORDER BY pa.book_count DESC;
```

**Exercise 4.5: Indexing & Performance**

(a) Create a large table (1M+ rows).
(b) Run a slow query without indexes.
(c) Add indexes on filter and join columns.
(d) Re-run with EXPLAIN ANALYZE and compare costs.

**Exercise 4.6: MongoDB CRUD & Aggregation**

(a) Create a `products` collection in MongoDB.
(b) Insert documents with nested fields (e.g., `reviews: [{rating, text}, ...]`).
(c) Write CRUD operations (insert, find, update, delete).
(d) Write an aggregation pipeline to get average rating per product and sort.

**Exercise 4.7: Redis Pub/Sub**

Write a pub/sub demo:
```python
import redis
import threading
import time

r = redis.Redis()

def publisher():
    for i in range(10):
        r.publish('events', f'Event {i}')
        time.sleep(1)

def subscriber():
    sub = r.pubsub()
    sub.subscribe('events')
    for msg in sub.listen():
        print(msg)

# Run in threads
threading.Thread(target=publisher).start()
threading.Thread(target=subscriber).start()
```

---

### **Advanced**

**Exercise 4.8: Zero-Downtime Migration**

Scenario: Add a `status` column (NOT NULL) to a 1M-row table without downtime.

Steps:
1. Add column with default: `ALTER TABLE products ADD COLUMN status VARCHAR DEFAULT 'active'`.
2. Create migration script to backfill in batches.
3. Make NOT NULL: `ALTER TABLE products ALTER COLUMN status SET NOT NULL`.
4. Verify no null values remain.

**Exercise 4.9: Detect & Fix N+1 Query**

Write an ORM query (SQLAlchemy/Django ORM) that causes N+1:
```python
# Bad
authors = Author.query.all()
for author in authors:
    book_count = len(author.books)  # N+1: query per author
```

Fix it with eager loading:
```python
# Good
authors = Author.query.options(joinedload(Author.books)).all()
```

Profile with SQLAlchemy echo or Django Debug Toolbar to confirm reduction in queries.

**Exercise 4.10: pgvector & Semantic Search**

(a) Install pgvector.
(b) Create a documents table with embeddings.
(c) Insert sample vectors (fake or from OpenAI).
(d) Write a query to find top-5 similar documents.
(e) Create an HNSW index and re-run; compare speed.

**Exercise 4.11: Great Expectations Data Quality Suite**

(a) Create a CSV with sample data (some bad rows: nulls, invalid values).
(b) Define expectations: no nulls in email, age between 18-100, status in ['active', 'inactive'].
(c) Run validation and generate a report.
(d) Fix the CSV and re-validate.

**Exercise 4.12: DynamoDB Single-Table Design**

Design a single table for an e-commerce app with 5 access patterns:
1. Get user profile.
2. Get all orders by user and date.
3. Get all items in order.
4. Get all orders by status.
5. Get user by email.

Define PK, SK, GSI(s), and write queries for each pattern.

---

### **Capstone**

**Exercise 4.13: Polyglot Persistence Demo**

Build a FastAPI (or Spring) service:

**Backend:**
- **Postgres**: Users, products, orders (transactional).
- **Redis**: Cache (user profile, product listings), rate limiting, session tokens.
- **MongoDB**: Activity logs, user reviews, audit trail.
- **pgvector**: Semantic search over product descriptions.

**Requirements:**
1. Implement CRUD endpoints for users, products, orders.
2. Add caching: GET /users/:id first checks Redis, then Postgres.
3. Rate limit: redis INCR on user_id, return 429 if exceeded.
4. Log activities to MongoDB on every mutation.
5. Semantic search endpoint: /products/search?query=laptop returns top-5 using pgvector.
6. Schema migrations: Flyway scripts for Postgres schema.
7. Data quality: Great Expectations suite validates Postgres tables weekly.
8. Integration tests: Testcontainers for Postgres, Redis, Mongo; mock pgvector.
9. Load test: pgbench-style script that runs 100 concurrent user sessions; generate EXPLAIN ANALYZE report.
10. Document the choice of each database; which queries/workloads motivated it.

**Deliverables:**
- Fully working service (docker-compose for all DBs).
- At least 10 endpoints.
- Test suite with >80% coverage.
- Migration scripts + zero-downtime plan.
- Data quality report.
- Load test results (TPS, latency, slow queries).
- README explaining architecture and DB choices.

---

## 5. Additional Reading & Resources

**Books:**
- *Designing Data-Intensive Applications* by Martin Kleppmann (essential; covers distributed systems, consistency, scale).
- *SQL Performance Explained* by Markus Winand (indexing deep dive).
- *Use The Index, Luke!* (free online; practical index tuning).
- *SQL Antipatterns* by Bill Karwin (common mistakes and how to avoid them).

**Official Documentation:**
- PostgreSQL docs (https://www.postgresql.org/docs/)
- MongoDB University (https://university.mongodb.com/)
- DynamoDB Book by Alex DeBrie (https://www.dynamodbbook.com/)
- Redis University (https://university.redis.io/)

**Courses & Tutorials:**
- SQLZoo (interactive SQL tutorial).
- LeetCode database problems (SQL, design).
- HackerRank SQL challenges.
- Brent Ozar's SQL Server posts (insights apply to most DBs).
- Neo4j GraphAcademy.

**Tools:**
- DBeaver, TablePlus (GUI clients).
- pgAdmin (Postgres management UI).
- Mongo Compass (MongoDB GUI).
- Datagrip (JetBrains IDE).
- Explain Analyzer (visualize EXPLAIN plans).

---

## 6. Index

- **Topic 1**: Postgres setup (Docker, psql, GUI tools).
- **Topic 2**: Relational model (relations, keys, constraints).
- **Topic 3**: Data types (numeric, text, boolean, date/time, JSON, arrays, UUID).
- **Topic 4**: DDL (CREATE, ALTER, DROP, schemas, sequences).
- **Topic 5**: DML (INSERT, UPDATE, DELETE, UPSERT, RETURNING).
- **Topic 6**: SELECT fundamentals (projection, filtering, sorting, LIMIT, OFFSET, DISTINCT).
- **Topic 7**: Joins (INNER, LEFT, RIGHT, FULL, CROSS, self, LATERAL).
- **Topic 8**: Grouping (GROUP BY, HAVING, GROUPING SETS, ROLLUP, CUBE).
- **Topic 9**: Subqueries (scalar, correlated, EXISTS, IN).
- **Topic 10**: CTEs & recursive CTEs (WITH clause, loops, tree traversal).
- **Topic 11**: Window functions (ROW_NUMBER, RANK, LAG/LEAD, aggregates OVER, PARTITION BY, ORDER BY).
- **Topic 12**: Views & materialized views (creation, refresh, updatable).
- **Topic 13**: Indexes (B-tree, hash, GIN, GIST, BRIN, covering, partial, expression).
- **Topic 14**: EXPLAIN & query plans (reading plans, seq scan vs index scan, buffers).
- **Topic 15**: Transactions & ACID (atomicity, consistency, isolation, durability).
- **Topic 16**: Locks & deadlocks (row, table, advisory, FOR UPDATE, deadlock detection).
- **Topic 17**: Normalization & denormalization (1NF, 2NF, 3NF, BCNF, intentional denormalization).
- **Topic 18**: Modeling patterns (lookup tables, audit, soft delete, SCD type 2, temporal).
- **Topic 19**: JSON/JSONB (operators, indexing, aggregation, JSONPath).
- **Topic 20**: PL/pgSQL (functions, procedures, triggers).
- **Topic 21**: Query tuning (collect, isolate, EXPLAIN, fix, measure).
- **Topic 22**: Security (roles, GRANT/REVOKE, RLS, SQL injection, parameterized queries).
- **Topic 23**: CAP theorem & PACELC (consistency, availability, partition-tolerance).
- **Topic 24**: Redis (strings, hashes, lists, sets, sorted sets, streams, TTL, pub/sub, persistence).
- **Topic 25**: MongoDB (BSON, CRUD, aggregation pipeline, indexes, schema design).
- **Topic 26**: Cassandra & DynamoDB (wide-column, partition/clustering keys, query-first design).
- **Topic 27**: DynamoDB (GSI, LSI, single-table design, capacity modes, TTL).
- **Topic 28**: Neo4j (Cypher, nodes, relationships, graph traversal).
- **Topic 29**: Time-series DBs (InfluxDB, Timescale, compression, downsampling).
- **Topic 30**: Elasticsearch (inverted index, analyzers, mappings, queries, aggregations).
- **Topic 31**: Vector DBs (pgvector, Pinecone, Milvus, embeddings, similarity search, HNSW).
- **Topic 32**: Polyglot persistence (choosing the right DB, common stacks, splitting strategies).
- **Topic 33**: DB layer testing (schema, constraints, data quality, migrations, performance, replication).
- **Topic 34**: Unit testing DB code (in-memory H2, Testcontainers, real engines).
- **Topic 35**: Migrations (Flyway, Liquibase, zero-downtime, up/down safety).
- **Topic 36**: Seed & teardown (transactional rollback, seed files, fixtures).
- **Topic 37**: Schema contract tests (column types, nullability, constraints).
- **Topic 38**: Data quality (Great Expectations, dbt tests, Soda SQL).
- **Topic 39**: Performance testing (pgbench, N+1 queries, slow-query logs).
- **Topic 40**: Replication & backup (streaming replication, failover, restore verification).
- **Topic 41**: PII & GDPR (anonymization, synthesis, test data policies).
- **Topic 42**: NoSQL testing (Testcontainers for Mongo, DynamoDB Local, Redis tests).
- **Topic 43**: Interview questions & gotchas (SQL vs NoSQL, optimization, isolation levels, CAP, denormalization, JOINs, N+1, indexes, locking).

---

## 7. References

1. Kleppmann, M. *Designing Data-Intensive Applications*. O'Reilly, 2017.
2. Winand, M. *SQL Performance Explained*. Self-published, 2014.
3. Karwin, B. *SQL Antipatterns*. Pragmatic Programmers, 2010.
4. DeBrie, A. *The DynamoDB Book*. Self-published, 2020.
5. PostgreSQL Documentation. https://www.postgresql.org/docs/
6. MongoDB University. https://university.mongodb.com/
7. Redis University. https://university.redis.io/
8. Neo4j GraphAcademy. https://graphacademy.neo4j.com/
9. DBeaver Documentation. https://dbeaver.io/
10. Brent Ozar's SQL Server Blog. https://www.brentozar.com/

---

**End of Document**

Subject 7 complete. Total estimated effort: 70 hours (30 SQL, 20 NoSQL, 20 testing & design). Mastering this subject will prepare you to discuss databases fluently in interviews and design data layers confidently.

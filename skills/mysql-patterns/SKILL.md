---
name: mysql-patterns
description: MySQL 8.0+ database patterns for query optimization, schema design, indexing, and security. Based on official MySQL documentation.
---

# MySQL 8.0+ Patterns

Quick reference for MySQL 8.0+ best practices. Based on official MySQL documentation at https://dev.mysql.com/doc/refman/8.0/en/

## When to Activate

- Writing SQL queries or migrations for MySQL 8.0+
- Designing database schemas for MySQL
- Troubleshooting slow queries
- Setting up prepared statements for security
- Configuring Performance Schema monitoring
- Optimizing indexes and data types

## Quick Reference

### Index Cheat Sheet

| Query Pattern | Index Type | Example |
|--------------|------------|---------|
| `WHERE col = value` | B-tree (default) | `CREATE INDEX idx ON t (col)` |
| `WHERE col > value` | B-tree | `CREATE INDEX idx ON t (col)` |
| `WHERE a = x AND b > y` | Composite | `CREATE INDEX idx ON t (a, b)` |
| `WHERE MATCH(text) AGAINST('query')` | FULLTEXT | `CREATE FULLTEXT INDEX idx ON t (col)` |
| `WHERE MBRContains(geom, point)` | SPATIAL | `CREATE SPATIAL INDEX idx ON t (col)` |
| `WHERE col IN (SELECT...)` | Hash (implicit) | Memory-based hash index |

**Key Principles:**
- B-tree indexes support equality and range comparisons on columns
- InnoDB uses B-tree indexes for all indexes except spatial indexes
- FULLTEXT indexes support natural-language text search (MyISAM and InnoDB)
- SPATIAL indexes optimize geometric queries (InnoDB only)

### Data Type Quick Reference

| Use Case | Correct Type | Avoid |
|----------|-------------|-------|
| IDs | `BIGINT UNSIGNED` | `INT`, `UUID` (for PK) |
| Strings (variable) | `VARCHAR(n)` | `TEXT` (for indexed columns) |
| Strings (fixed) | `CHAR(n)` | `VARCHAR` (for known-length data) |
| Timestamps | `TIMESTAMP` | `DATETIME` (for timezone-aware) |
| Money | `DECIMAL(19,4)` | `FLOAT`, `DOUBLE` |
| Flags | `BOOLEAN` | `TINYINT`, `VARCHAR` |
| JSON | `JSON` | `TEXT` (for JSON data) |
| IP addresses | `INT UNSIGNED` (IPv4) | `VARCHAR(15)` |

**Type Selection Guidelines:**
- Use the smallest data type that can reliably store your data
- Smaller types require less storage, memory, and CPU cache
- `TINYINT`: 0-255 (or -128 to 127 signed) - 1 byte
- `SMALLINT`: 0-65,535 (or -32,768 to 32,767) - 2 bytes
- `MEDIUMINT`: 0-16,777,215 - 3 bytes
- `INT`: 0-2,147,483,647 - 4 bytes
- `BIGINT`: 0-18,446,744,073,709,551,615 - 8 bytes
- Use `UNSIGNED` to double positive range when negative values aren't needed

### Common Patterns

**Composite Index Order (Leftmost Prefix):**
```sql
-- MySQL can use multi-column indexes for queries testing all columns,
-- or queries testing just the first column, first two columns, etc.
CREATE INDEX idx_name ON users (last_name, first_name);

-- Works: Uses last_name part of index
SELECT * FROM users WHERE last_name = 'Smith';

-- Works: Uses both parts of index
SELECT * FROM users WHERE last_name = 'Smith' AND first_name = 'John';

-- Does NOT work: Cannot use index for first_name alone
SELECT * FROM users WHERE first_name = 'John';

-- Best practice: Equality columns first, then range columns
CREATE INDEX idx_order ON orders (status, created_at);
-- Works for: WHERE status = 'pending' AND created_at > '2024-01-01'
```

**Covering Index:**
```sql
-- InnoDB secondary indexes automatically include primary key columns
-- A covering index includes all columns retrieved by a query
CREATE INDEX idx_covering ON orders (user_id, status) INCLUDE (created_at, total);

-- Query avoids table lookup (Using index in EXPLAIN)
SELECT user_id, status, created_at, total
FROM orders
WHERE user_id = 123 AND status = 'completed';
```

**Multiple-Column Index Strategy:**
```sql
-- Index up to 16 columns (MySQL limit)
-- Column order is critical for query optimization
CREATE INDEX idx_composite ON products (
  category_id,      -- Equality filter
  brand_id,         -- Equality filter
  price             -- Range filter or sort
);

-- Optimal query pattern:
SELECT * FROM products
WHERE category_id = 5
  AND brand_id = 12
  AND price > 100
ORDER BY price;
```

**Prepared Statements (Security):**
```sql
-- Prevents SQL injection by separating SQL logic from data
PREPARE stmt FROM 'SELECT * FROM users WHERE email = ? AND status = ?');
SET @email = 'user@example.com';
SET @status = 'active';
EXECUTE stmt USING @email, @status;
DEALLOCATE PREPARE stmt;

-- Application example (parameter binding varies by language)
-- PHP: mysqli_prepare(), PDO::prepare()
-- Java: PreparedStatement
-- Python: cursor.execute(query, params)
-- Node.js: connection.execute(query, params)
```

**Cursor/Keyset Pagination:**
```sql
-- More efficient than OFFSET for large datasets
SELECT * FROM products
WHERE id > last_seen_id
ORDER BY id
LIMIT 20;

-- Composite key pagination
SELECT * FROM orders
WHERE (user_id, created_at) > (:user_id, :created_at)
ORDER BY user_id, created_at
LIMIT 20;

-- Avoid OFFSET with large values (O(n) performance)
-- Use WHERE with indexed column instead
```

**Index Condition Pushdown (ICP):**
```sql
-- MySQL 5.6+ uses ICP to reduce storage engine access
-- Index filters conditions at storage engine level
CREATE INDEX idx_icp ON orders (status, created_at);

-- Query benefits from ICP
SELECT * FROM orders
WHERE status = 'pending'
  AND created_at > '2024-01-01'
  AND total > 1000;  -- Filter applied at index level
```

### Performance Monitoring

**Slow Query Log:**
```sql
-- Enable slow query log
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 2;  -- Seconds
SET GLOBAL log_queries_not_using_indexes = 'ON';

-- MySQL 8.0.14+: Enable extra fields
SET GLOBAL log_slow_extra = 'ON';

-- Check configuration
SHOW VARIABLES LIKE '%slow_query%';

-- Output format:
-- # Time: 2024-01-15T10:30:00.123456Z
-- # User@Host: app[app] @ localhost []
-- # Query_time: 2.345678  Lock_time: 0.000123 Rows_sent: 10  Rows_examined: 50000
-- SET timestamp=1705315800;
-- SELECT * FROM orders WHERE status = 'pending';
```

**EXPLAIN Analysis:**
```sql
-- Basic usage
EXPLAIN SELECT * FROM users WHERE email = 'user@example.com';

-- Extended output (MySQL 8.0.18+)
EXPLAIN ANALYZE SELECT * FROM orders WHERE user_id = 123;

-- Key columns to check:
-- type: Join type (best to worst: system, const, eq_ref, ref, range, index, ALL)
-- possible_keys: Indexes MySQL can choose from
-- key: Actual index chosen
-- rows: Estimated rows to examine
-- Extra: Additional information (watch for "Using filesort", "Using temporary", "ALL")
```

**Performance Schema Queries:**
```sql
-- Enable Performance Schema (enabled by default in MySQL 8.0+)
UPDATE performance_schema.setup_instruments
SET ENABLED = 'YES', TIMED = 'YES'
WHERE NAME LIKE '%statement/%';

-- Find slow statements
SELECT DIGEST_TEXT, COUNT_STAR, AVG_TIMER_WAIT/1000000000000 AS avg_sec
FROM performance_schema.events_statements_summary_by_digest
ORDER BY AVG_TIMER_WAIT DESC
LIMIT 10;

-- Monitor table I/O
SELECT OBJECT_NAME, COUNT_READ, COUNT_WRITE
FROM performance_schema.table_io_waits_summary_by_table
WHERE OBJECT_SCHEMA = 'your_database'
ORDER BY COUNT_READ + COUNT_WRITE DESC;
```

### Anti-Pattern Detection

**Missing Indexes Detection:**
```sql
-- Find columns with no indexes in WHERE/JOIN clauses
-- Use Performance Schema or slow query log
SELECT OBJECT_SCHEMA, OBJECT_NAME, COUNT_STAR, SUM_TIMER_WAIT/1000000000000 AS total_sec
FROM performance_schema.events_waits_summary_by_table
WHERE EVENT_NAME LIKE '%wait/io/table/%'
ORDER BY SUM_TIMER_WAIT DESC;

-- Check for full table scans (type = ALL in EXPLAIN)
EXPLAIN SELECT * FROM large_table WHERE unindexed_column = 'value';
```

**Unused Indexes Detection:**
```sql
-- Find indexes that are never used
SELECT
  OBJECT_SCHEMA,
  OBJECT_NAME,
  INDEX_NAME
FROM performance_schema.table_io_waits_summary_by_index_usage
WHERE INDEX_NAME IS NOT NULL
  AND COUNT_STAR = 0
  AND INDEX_NAME != 'PRIMARY'
ORDER BY OBJECT_SCHEMA, OBJECT_NAME;
```

**Slow Query Analysis:**
```sql
-- Identify queries with high row examination
SELECT * FROM mysql.slow_log
WHERE rows_examined > 10000
ORDER BY query_time DESC
LIMIT 20;

-- Find queries with filesort
-- Run EXPLAIN and look for "Using filesort" in Extra column
-- This indicates ORDER BY cannot use the index
```

**InnoDB Lock Wait Analysis:**
```sql
-- Check for lock contention
SELECT * FROM performance_schema.events_waits_current
WHERE EVENT_NAME LIKE '%lock%'
  AND SOURCE IS NOT NULL;

-- Find transactions holding locks
SELECT * FROM performance_schema.data_locks
WHERE LOCKED_BY_THREAD_ID IS NOT NULL;
```

### Configuration Template

```ini
# my.cnf / my.ini (MySQL 8.0+)

# Performance Settings
[mysqld]
innodb_buffer_pool_size = 2G          # 70-80% of RAM for dedicated DB
innodb_log_file_size = 256M           # Large enough for write workload
innodb_flush_log_at_trx_commit = 2    # Performance over durability
innodb_flush_method = O_DIRECT

# Connection Settings
max_connections = 200                 # Adjust based on application needs
thread_cache_size = 16
table_open_cache = 4000

# Query Optimization
query_cache_size = 0                  # Disabled in MySQL 8.0+
query_cache_type = 0                  # Query cache removed in 8.0
tmp_table_size = 64M
max_heap_table_size = 64M

# Slow Query Log
slow_query_log = ON
long_query_time = 2
log_queries_not_using_indexes = ON
log_slow_extra = ON                   # MySQL 8.0.14+

# Performance Schema
performance_schema = ON

# Security
sql_mode = STRICT_TRANS_TABLES,NO_ZERO_DATE,NO_ZERO_IN_DATE,ERROR_FOR_DIVISION_BY_ZERO
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci

# InnoDB Settings
innodb_file_per_table = ON
innodb_stats_on_metadata = OFF
```

### Security Best Practices

**Prepared Statements for SQL Injection Prevention:**
```sql
-- Always use prepared statements for user input
-- SQL injection protection: Parameter values can contain unescaped
-- SQL quote and delimiter characters safely

-- Application-level prepared statements preferred (binary protocol)
-- SQL-level prepared statements available when needed
PREPARE stmt FROM 'SELECT * FROM users WHERE id = ?');
SET @user_id = 1;
EXECUTE stmt USING @user_id;
DEALLOCATE PREPARE stmt;
```

**Privilege Management (MySQL 8.0 Roles):**
```sql
-- Create roles for privilege management
CREATE ROLE 'app_read', 'app_write', 'app_admin';

-- Grant privileges to roles
GRANT SELECT ON app_db.* TO 'app_read';
GRANT SELECT, INSERT, UPDATE ON app_db.* TO 'app_write';
GRANT ALL PRIVILEGES ON app_db.* TO 'app_admin';

-- Grant roles to users
CREATE USER 'app_user'@'%' IDENTIFIED BY 'secure_password';
GRANT 'app_write' TO 'app_user'@'%';

-- Set default role
SET DEFAULT ROLE ALL TO 'app_user'@'%';

-- Revoke privileges
REVOKE 'app_admin' FROM 'app_user'@'%';
```

**Authentication Best Practices:**
```sql
-- Use strong authentication plugins
ALTER USER 'admin'@'localhost' IDENTIFIED WITH 'caching_sha2_password' BY 'strong_password';

-- Require password policy
INSTALL COMPONENT 'file://component_validate_password';
SHOW VARIABLES LIKE 'validate_password%';

-- Password expiration policy
ALTER USER 'app_user'@'%' PASSWORD EXPIRE INTERVAL 90 DAY;
```

### Performance Optimization Checklist

**Index Optimization:**
- [ ] Create indexes on all frequently used WHERE, JOIN, and ORDER BY columns
- [ ] Use composite indexes for multi-column queries (respect leftmost prefix)
- [ ] Monitor unused indexes and remove them
- [ ] Consider covering indexes for frequent queries
- [ ] Use appropriate index types (B-tree, FULLTEXT, SPATIAL)

**Query Optimization:**
- [ ] Use EXPLAIN to analyze query execution plans
- [ ] Avoid SELECT * - select only needed columns
- [ ] Use WHERE instead of HAVING for row filtering
- [ ] Use LIMIT for large result sets
- [ ] Avoid subqueries in WHERE clause - use JOIN instead
- [ ] Use UNION ALL instead of UNION (unless deduplication needed)

**Data Type Optimization:**
- [ ] Use smallest appropriate data types
- [ ] Use UNSIGNED for non-negative integers
- [ ] Use DATETIME/TIMESTAMP instead of string storage for dates
- [ ] Use DECIMAL for monetary values, not FLOAT/DOUBLE
- [ ] Normalize data to reduce redundancy

**Configuration Optimization:**
- [ ] Configure innodb_buffer_pool_size (70-80% of RAM)
- [ ] Enable and configure slow query log
- [ ] Enable Performance Schema for monitoring
- [ ] Set appropriate max_connections limit
- [ ] Configure innodb_flush_log_at_trx_commit based on durability needs

## Related

- Official MySQL 8.0 Documentation: https://dev.mysql.com/doc/refman/8.0/en/
- MySQL Performance Schema: https://dev.mysql.com/doc/mysql-perfschema-excerpt/8.0/en/
- MySQL 8.0 Security Guide: https://dev.mysql.com/doc/mysql-secure-deployment-guide/8.0/en/
- Skill: `postgres-patterns` - PostgreSQL patterns
- Skill: `backend-patterns` - API and backend patterns

---

*Based on [MySQL 8.0 Reference Manual](https://dev.mysql.com/doc/refman/8.0/en/) and official MySQL documentation*

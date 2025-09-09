# PostgreSQL Cheat Sheet

Comprehensive reference for PostgreSQL database management system - covering advanced features, JSON operations, full-text search, and administration for production environments.

---

## Table of Contents
- [Installation & Connection](#installation--connection)
- [Basic Operations](#basic-operations)
- [Data Types](#data-types)
- [Advanced Features](#advanced-features)
- [JSON Operations](#json-operations)
- [Full-Text Search](#full-text-search)
- [Indexes & Performance](#indexes--performance)
- [Window Functions](#window-functions)
- [Common Table Expressions (CTEs)](#common-table-expressions-ctes)
- [Administration](#administration)
- [Stored Procedures & Functions](#stored-procedures--functions)
- [Backup & Recovery](#backup--recovery)
- [Monitoring & Optimization](#monitoring--optimization)
- [Extensions](#extensions)
- [Integration Patterns](#integration-patterns)

---

## Installation & Connection

### Installation
```bash
# Ubuntu/Debian
sudo apt update && sudo apt install postgresql postgresql-contrib

# CentOS/RHEL
sudo yum install postgresql-server postgresql-contrib
sudo postgresql-setup initdb

# macOS (Homebrew)
brew install postgresql
brew services start postgresql

# Docker
docker run --name postgres-container -e POSTGRES_PASSWORD=password -d postgres:15
```

### Connection
```bash
# Command line (psql)
psql -h hostname -U username -d database_name

# Local connection
psql -U postgres

# Connection with specific parameters
psql -h localhost -p 5432 -U myuser -d mydb

# Connection string
psql "postgresql://username:password@hostname:5432/database_name"
```

### Connection Parameters
| Parameter | Description | Example |
|-----------|-------------|---------|
| `-h` | Hostname | `psql -h localhost` |
| `-p` | Port | `psql -p 5432` |
| `-U` | Username | `psql -U postgres` |
| `-d` | Database | `psql -d mydb` |
| `-W` | Force password prompt | `psql -W` |
| `-c` | Execute command | `psql -c "SELECT version()"` |

---

## Basic Operations

### Database Operations
```sql
-- Create database
CREATE DATABASE myapp WITH ENCODING 'UTF8' LC_COLLATE='en_US.UTF-8' LC_CTYPE='en_US.UTF-8';

-- List databases
\l
-- or
SELECT datname FROM pg_database;

-- Connect to database
\c myapp

-- Drop database
DROP DATABASE myapp;

-- Current database
SELECT current_database();
```

### Schema Operations
```sql
-- Create schema
CREATE SCHEMA sales;

-- List schemas
\dn
-- or
SELECT schema_name FROM information_schema.schemata;

-- Set search path
SET search_path TO sales, public;

-- Drop schema
DROP SCHEMA sales CASCADE;
```

### Table Operations
```sql
-- Create table with constraints
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    profile JSONB,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- List tables
\dt
-- or
SELECT table_name FROM information_schema.tables WHERE table_schema = 'public';

-- Describe table
\d users
-- or
SELECT column_name, data_type, is_nullable 
FROM information_schema.columns 
WHERE table_name = 'users';

-- Alter table
ALTER TABLE users ADD COLUMN phone VARCHAR(20);
ALTER TABLE users DROP COLUMN phone;
ALTER TABLE users ALTER COLUMN username TYPE VARCHAR(60);

-- Add constraints
ALTER TABLE users ADD CONSTRAINT check_email CHECK (email ~ '^[^@]+@[^@]+\.[^@]+$');

-- Drop table
DROP TABLE users;
```

### CRUD Operations
```sql
-- INSERT
INSERT INTO users (username, email, profile) 
VALUES ('john_doe', 'john@example.com', '{"age": 30, "city": "NYC"}');

-- Multiple inserts
INSERT INTO users (username, email) VALUES 
    ('jane_doe', 'jane@example.com'),
    ('bob_smith', 'bob@example.com');

-- INSERT with ON CONFLICT (UPSERT)
INSERT INTO users (username, email) 
VALUES ('john_doe', 'john@example.com')
ON CONFLICT (username) 
DO UPDATE SET email = EXCLUDED.email, updated_at = CURRENT_TIMESTAMP;

-- SELECT
SELECT * FROM users;
SELECT username, email FROM users WHERE id = 1;

-- UPDATE
UPDATE users SET email = 'newemail@example.com' WHERE id = 1;

-- DELETE
DELETE FROM users WHERE id = 1;

-- TRUNCATE
TRUNCATE TABLE users RESTART IDENTITY;
```

---

## Data Types

### Numeric Types
| Type | Size | Range | Usage |
|------|------|-------|-------|
| `SMALLINT` | 2 bytes | -32,768 to 32,767 | Small integers |
| `INTEGER` | 4 bytes | -2B to 2B | Standard integers |
| `BIGINT` | 8 bytes | Large range | Large integers |
| `SERIAL` | 4 bytes | Auto-increment | Primary keys |
| `BIGSERIAL` | 8 bytes | Auto-increment | Large primary keys |
| `DECIMAL(p,s)` | Variable | Exact precision | Financial calculations |
| `NUMERIC(p,s)` | Variable | Exact precision | Same as DECIMAL |
| `REAL` | 4 bytes | 6 decimal digits | Single precision |
| `DOUBLE PRECISION` | 8 bytes | 15 decimal digits | Double precision |

### String Types
```sql
-- Character types
CHAR(n)           -- Fixed length
VARCHAR(n)        -- Variable length with limit
TEXT              -- Variable unlimited length

-- Example
CREATE TABLE examples (
    code CHAR(10),           -- Fixed 10 characters
    name VARCHAR(100),       -- Up to 100 characters
    description TEXT         -- Unlimited length
);
```

### Date/Time Types
```sql
-- Date/time types
DATE                    -- Date only
TIME                    -- Time only
TIMESTAMP              -- Date and time
TIMESTAMPTZ            -- Timestamp with timezone
INTERVAL               -- Time interval

-- Examples
CREATE TABLE events (
    id SERIAL PRIMARY KEY,
    event_name TEXT,
    event_date DATE,
    event_time TIME,
    created_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP,
    duration INTERVAL
);

-- Date/time operations
SELECT NOW();                                    -- Current timestamp with timezone
SELECT CURRENT_DATE;                            -- Current date
SELECT CURRENT_TIME;                            -- Current time
SELECT CURRENT_TIMESTAMP;                       -- Current timestamp
SELECT NOW() + INTERVAL '1 day';               -- Tomorrow
SELECT NOW() - INTERVAL '1 week';              -- A week ago
SELECT AGE(NOW(), '1990-01-01');               -- Age calculation
SELECT EXTRACT(YEAR FROM NOW());               -- Extract year
```

### JSON/JSONB Types
```sql
-- JSON vs JSONB
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    data JSON,           -- Stores exact copy, slower queries
    metadata JSONB       -- Binary format, faster queries, supports indexing
);

-- JSONB is generally preferred for most use cases
INSERT INTO products (metadata) VALUES 
('{"name": "Laptop", "price": 999, "specs": {"cpu": "Intel i7", "ram": "16GB"}}');
```

### Array Types
```sql
-- Array columns
CREATE TABLE articles (
    id SERIAL PRIMARY KEY,
    title TEXT,
    tags TEXT[],                    -- Array of text
    ratings INTEGER[]               -- Array of integers
);

-- Insert arrays
INSERT INTO articles (title, tags, ratings) VALUES 
('PostgreSQL Guide', ARRAY['database', 'sql', 'postgres'], ARRAY[5, 4, 5]);

-- Query arrays
SELECT * FROM articles WHERE 'database' = ANY(tags);
SELECT * FROM articles WHERE tags @> ARRAY['sql'];     -- Contains
SELECT * FROM articles WHERE tags && ARRAY['python'];   -- Overlaps
```

---

## Advanced Features

### UPSERT (ON CONFLICT)
```sql
-- Basic upsert
INSERT INTO users (id, username, email) 
VALUES (1, 'john_doe', 'john@example.com')
ON CONFLICT (id) 
DO UPDATE SET 
    username = EXCLUDED.username,
    email = EXCLUDED.email,
    updated_at = CURRENT_TIMESTAMP;

-- Conditional upsert
INSERT INTO products (sku, name, price) 
VALUES ('ABC123', 'New Product', 99.99)
ON CONFLICT (sku) 
DO UPDATE SET 
    price = EXCLUDED.price
WHERE products.price != EXCLUDED.price;

-- Do nothing on conflict
INSERT INTO users (username, email) 
VALUES ('existing_user', 'email@example.com')
ON CONFLICT (username) DO NOTHING;
```

### LATERAL Joins
```sql
-- LATERAL join allows correlated subqueries in FROM clause
SELECT u.username, recent_posts.title, recent_posts.created_at
FROM users u
CROSS JOIN LATERAL (
    SELECT title, created_at
    FROM posts p
    WHERE p.user_id = u.id
    ORDER BY created_at DESC
    LIMIT 3
) AS recent_posts;
```

### Recursive Queries
```sql
-- Recursive CTE for hierarchical data
WITH RECURSIVE employee_hierarchy AS (
    -- Base case: top-level managers
    SELECT id, name, manager_id, 0 as level
    FROM employees
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- Recursive case
    SELECT e.id, e.name, e.manager_id, eh.level + 1
    FROM employees e
    JOIN employee_hierarchy eh ON e.manager_id = eh.id
)
SELECT * FROM employee_hierarchy ORDER BY level, name;

-- Generate series
SELECT * FROM generate_series(1, 10);                    -- Numbers 1-10
SELECT * FROM generate_series('2023-01-01'::date, '2023-01-31'::date, '1 day');
```

### GENERATED Columns
```sql
-- Computed columns (PostgreSQL 12+)
CREATE TABLE rectangles (
    id SERIAL PRIMARY KEY,
    width NUMERIC,
    height NUMERIC,
    area NUMERIC GENERATED ALWAYS AS (width * height) STORED
);

INSERT INTO rectangles (width, height) VALUES (10, 5);
SELECT * FROM rectangles;  -- area will be automatically calculated as 50
```

---

## JSON Operations

### JSONB Operators
```sql
-- JSON operators
SELECT data->>'name' AS name FROM products;              -- Extract as text
SELECT data->'specs'->>'cpu' AS cpu FROM products;       -- Nested extraction
SELECT data->'specs' AS specs FROM products;             -- Extract as JSON
SELECT data#>'{specs, cpu}' AS cpu FROM products;        -- Path extraction
SELECT data ? 'name' AS has_name FROM products;          -- Key exists
SELECT data ?& ARRAY['name', 'price'] FROM products;     -- All keys exist
SELECT data ?| ARRAY['name', 'title'] FROM products;     -- Any key exists
SELECT data @> '{"category": "electronics"}' FROM products;  -- Contains
SELECT data <@ '{"name": "Laptop", "price": 999}' FROM products;  -- Contained by
```

### JSONB Functions
```sql
-- JSONB manipulation functions
SELECT jsonb_set(data, '{price}', '1099') FROM products; -- Set value
SELECT data || '{"category": "electronics"}' FROM products; -- Concatenate
SELECT data - 'price' FROM products;                     -- Remove key
SELECT data #- '{specs, ram}' FROM products;            -- Remove nested key
SELECT jsonb_array_elements(data->'tags') FROM products; -- Expand array
SELECT jsonb_object_keys(data) FROM products;           -- Get all keys
SELECT jsonb_each(data) FROM products;                  -- Get key-value pairs

-- JSONB aggregation
SELECT jsonb_agg(data) FROM products;                   -- Aggregate to array
SELECT jsonb_object_agg(name, price) FROM products;     -- Aggregate to object
```

### JSONB Indexing
```sql
-- Create indexes on JSONB columns
CREATE INDEX idx_product_name ON products USING GIN ((data->>'name'));
CREATE INDEX idx_product_data ON products USING GIN (data);

-- Partial index on JSONB
CREATE INDEX idx_active_products ON products USING GIN (data) 
WHERE data->>'status' = 'active';

-- Query optimization with JSONB indexes
EXPLAIN (ANALYZE, BUFFERS) 
SELECT * FROM products WHERE data->>'category' = 'electronics';
```

---

## Full-Text Search

### Basic Text Search
```sql
-- Simple text search
SELECT * FROM articles 
WHERE to_tsvector('english', title || ' ' || content) @@ to_tsquery('english', 'postgresql');

-- Using plainto_tsquery for natural language
SELECT * FROM articles 
WHERE to_tsvector('english', title || ' ' || content) @@ plainto_tsquery('english', 'postgresql database');

-- Phrase search
SELECT * FROM articles 
WHERE to_tsvector('english', title || ' ' || content) @@ phraseto_tsquery('english', 'postgresql database');
```

### Text Search Configuration
```sql
-- Create text search column
ALTER TABLE articles ADD COLUMN search_vector tsvector;

-- Update search vector
UPDATE articles SET search_vector = 
    to_tsvector('english', coalesce(title, '') || ' ' || coalesce(content, ''));

-- Create GIN index for full-text search
CREATE INDEX idx_articles_search ON articles USING GIN (search_vector);

-- Trigger to maintain search vector
CREATE OR REPLACE FUNCTION update_search_vector() RETURNS trigger AS $$
BEGIN
    NEW.search_vector := to_tsvector('english', coalesce(NEW.title, '') || ' ' || coalesce(NEW.content, ''));
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER update_articles_search_vector
    BEFORE INSERT OR UPDATE ON articles
    FOR EACH ROW EXECUTE FUNCTION update_search_vector();
```

### Advanced Text Search
```sql
-- Ranking results
SELECT title, 
       ts_rank(search_vector, to_tsquery('english', 'postgresql')) as rank
FROM articles 
WHERE search_vector @@ to_tsquery('english', 'postgresql')
ORDER BY rank DESC;

-- Highlighting results
SELECT title,
       ts_headline('english', content, to_tsquery('english', 'postgresql')) as highlight
FROM articles 
WHERE search_vector @@ to_tsquery('english', 'postgresql');

-- Boolean search operators
SELECT * FROM articles 
WHERE search_vector @@ to_tsquery('english', 'postgresql & database');     -- AND
SELECT * FROM articles 
WHERE search_vector @@ to_tsquery('english', 'postgresql | mysql');        -- OR
SELECT * FROM articles 
WHERE search_vector @@ to_tsquery('english', 'database & !mysql');         -- NOT
```

---

## Indexes & Performance

### Index Types
```sql
-- B-tree index (default)
CREATE INDEX idx_username ON users (username);

-- Unique index
CREATE UNIQUE INDEX idx_email ON users (email);

-- Partial index
CREATE INDEX idx_active_users ON users (username) WHERE status = 'active';

-- Multi-column index
CREATE INDEX idx_user_email ON users (username, email);

-- Expression index
CREATE INDEX idx_lower_email ON users (LOWER(email));

-- GIN index (for arrays, JSONB, full-text search)
CREATE INDEX idx_tags ON articles USING GIN (tags);
CREATE INDEX idx_content ON articles USING GIN (to_tsvector('english', content));

-- GiST index (for geometric data, full-text search)
CREATE INDEX idx_location ON stores USING GIST (location);

-- Hash index (equality comparisons only)
CREATE INDEX idx_user_hash ON users USING HASH (user_id);

-- BRIN index (for large, naturally ordered tables)
CREATE INDEX idx_created_at_brin ON logs USING BRIN (created_at);
```

### Index Management
```sql
-- List indexes
\di
-- or
SELECT indexname, tablename, indexdef 
FROM pg_indexes 
WHERE schemaname = 'public';

-- Index usage statistics
SELECT schemaname, tablename, indexname, idx_scan, idx_tup_read, idx_tup_fetch
FROM pg_stat_user_indexes
ORDER BY idx_scan DESC;

-- Unused indexes
SELECT schemaname, tablename, indexname, idx_scan
FROM pg_stat_user_indexes
WHERE idx_scan = 0 AND schemaname = 'public';

-- Drop index
DROP INDEX idx_username;

-- Reindex
REINDEX INDEX idx_username;
REINDEX TABLE users;
```

### Query Optimization
```sql
-- EXPLAIN plans
EXPLAIN SELECT * FROM users WHERE username = 'john_doe';
EXPLAIN (ANALYZE, BUFFERS) SELECT * FROM users WHERE username = 'john_doe';
EXPLAIN (ANALYZE, BUFFERS, FORMAT JSON) SELECT * FROM users WHERE username = 'john_doe';

-- Analyze table statistics
ANALYZE users;

-- Vacuum and analyze
VACUUM ANALYZE users;

-- Auto-vacuum settings
SELECT name, setting FROM pg_settings WHERE name LIKE 'autovacuum%';
```

---

## Window Functions

### Basic Window Functions
```sql
-- Row number
SELECT username, email, 
       ROW_NUMBER() OVER (ORDER BY created_at) as row_num
FROM users;

-- Rank functions
SELECT username, score,
       RANK() OVER (ORDER BY score DESC) as rank,
       DENSE_RANK() OVER (ORDER BY score DESC) as dense_rank,
       PERCENT_RANK() OVER (ORDER BY score DESC) as percent_rank
FROM user_scores;

-- Lag and Lead
SELECT username, created_at,
       LAG(created_at) OVER (ORDER BY created_at) as prev_created,
       LEAD(created_at) OVER (ORDER BY created_at) as next_created
FROM users;
```

### Window Functions with Partitions
```sql
-- Partition by department
SELECT name, department, salary,
       AVG(salary) OVER (PARTITION BY department) as dept_avg,
       RANK() OVER (PARTITION BY department ORDER BY salary DESC) as dept_rank
FROM employees;

-- Running totals
SELECT date, amount,
       SUM(amount) OVER (ORDER BY date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) as running_total
FROM transactions;

-- Moving average
SELECT date, amount,
       AVG(amount) OVER (ORDER BY date ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) as moving_avg_3
FROM daily_sales;
```

### Advanced Window Functions
```sql
-- First and last values
SELECT username, created_at,
       FIRST_VALUE(username) OVER (ORDER BY created_at) as first_user,
       LAST_VALUE(username) OVER (ORDER BY created_at ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) as last_user
FROM users;

-- Nth value
SELECT username, score,
       NTH_VALUE(username, 2) OVER (ORDER BY score DESC) as second_highest
FROM user_scores;

-- Cumulative distribution
SELECT username, score,
       CUME_DIST() OVER (ORDER BY score) as cumulative_dist
FROM user_scores;
```

---

## Common Table Expressions (CTEs)

### Basic CTE
```sql
-- Simple CTE
WITH high_value_customers AS (
    SELECT customer_id, SUM(amount) as total_spent
    FROM orders
    GROUP BY customer_id
    HAVING SUM(amount) > 1000
)
SELECT c.name, hvc.total_spent
FROM customers c
JOIN high_value_customers hvc ON c.id = hvc.customer_id;
```

### Multiple CTEs
```sql
-- Multiple CTEs
WITH 
customer_totals AS (
    SELECT customer_id, SUM(amount) as total_spent
    FROM orders
    GROUP BY customer_id
),
average_spending AS (
    SELECT AVG(total_spent) as avg_total
    FROM customer_totals
)
SELECT c.name, ct.total_spent, 
       CASE 
           WHEN ct.total_spent > a.avg_total THEN 'Above Average'
           ELSE 'Below Average'
       END as spending_category
FROM customers c
JOIN customer_totals ct ON c.id = ct.customer_id
CROSS JOIN average_spending a;
```

### Recursive CTE
```sql
-- Generate date series
WITH RECURSIVE date_series AS (
    SELECT '2023-01-01'::date as date
    UNION ALL
    SELECT date + INTERVAL '1 day'
    FROM date_series
    WHERE date < '2023-01-31'::date
)
SELECT date FROM date_series;

-- Organizational hierarchy
WITH RECURSIVE org_chart AS (
    SELECT id, name, manager_id, 0 as level, name as path
    FROM employees
    WHERE manager_id IS NULL
    
    UNION ALL
    
    SELECT e.id, e.name, e.manager_id, oc.level + 1, oc.path || ' -> ' || e.name
    FROM employees e
    JOIN org_chart oc ON e.manager_id = oc.id
)
SELECT * FROM org_chart ORDER BY level, name;
```

---

## Administration

### User Management
```sql
-- Create user
CREATE USER appuser WITH PASSWORD 'password';

-- Create user with additional options
CREATE USER readonly WITH PASSWORD 'password' 
    VALID UNTIL '2024-12-31' 
    CONNECTION LIMIT 10;

-- Alter user
ALTER USER appuser WITH PASSWORD 'newpassword';
ALTER USER appuser VALID UNTIL '2025-12-31';

-- Grant privileges
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO appuser;
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA public TO appuser;
GRANT EXECUTE ON ALL FUNCTIONS IN SCHEMA public TO appuser;

-- Create role
CREATE ROLE app_readers;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO app_readers;
GRANT app_readers TO appuser;

-- List users and roles
\du
-- or
SELECT usename, usesuper, usecreatedb, useconnlimit 
FROM pg_user;

-- Drop user
DROP USER appuser;
```

### Database Configuration
```sql
-- Show configuration
SHOW all;
SHOW shared_buffers;
SHOW max_connections;

-- Set parameters
SET work_mem = '256MB';
SET shared_buffers = '1GB';

-- Configuration file locations
SHOW config_file;
SHOW hba_file;
SHOW data_directory;
```

### Monitoring Connections
```sql
-- Current connections
SELECT pid, usename, application_name, client_addr, state, query_start, query
FROM pg_stat_activity
WHERE state = 'active';

-- Kill connection
SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE pid = 12345;

-- Database statistics
SELECT datname, numbackends, xact_commit, xact_rollback, 
       blks_read, blks_hit, temp_files, temp_bytes
FROM pg_stat_database;
```

### Maintenance Tasks
```sql
-- Manual vacuum
VACUUM users;
VACUUM FULL users;  -- More thorough but locks table

-- Analyze statistics
ANALYZE users;

-- Reindex
REINDEX TABLE users;
REINDEX INDEX idx_username;

-- Check table bloat
SELECT schemaname, tablename, 
       pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) as size,
       pg_size_pretty(pg_relation_size(schemaname||'.'||tablename)) as table_size,
       pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename) - pg_relation_size(schemaname||'.'||tablename)) as index_size
FROM pg_tables 
WHERE schemaname = 'public'
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;
```

---

## Stored Procedures & Functions

### Functions
```sql
-- Basic function
CREATE OR REPLACE FUNCTION get_user_count()
RETURNS INTEGER AS $$
BEGIN
    RETURN (SELECT COUNT(*) FROM users);
END;
$$ LANGUAGE plpgsql;

-- Function with parameters
CREATE OR REPLACE FUNCTION get_posts_by_user(user_id INTEGER)
RETURNS TABLE(id INTEGER, title TEXT, created_at TIMESTAMPTZ) AS $$
BEGIN
    RETURN QUERY
    SELECT p.id, p.title, p.created_at
    FROM posts p
    WHERE p.user_id = get_posts_by_user.user_id
    ORDER BY p.created_at DESC;
END;
$$ LANGUAGE plpgsql;

-- Function returning JSON
CREATE OR REPLACE FUNCTION get_user_profile(user_id INTEGER)
RETURNS JSON AS $$
BEGIN
    RETURN (
        SELECT json_build_object(
            'id', id,
            'username', username,
            'email', email,
            'created_at', created_at
        )
        FROM users 
        WHERE id = user_id
    );
END;
$$ LANGUAGE plpgsql;

-- Call functions
SELECT get_user_count();
SELECT * FROM get_posts_by_user(1);
SELECT get_user_profile(1);
```

### Procedures (PostgreSQL 11+)
```sql
-- Stored procedure
CREATE OR REPLACE PROCEDURE transfer_funds(
    sender_id INTEGER,
    receiver_id INTEGER,
    amount DECIMAL
) AS $$
BEGIN
    -- Start transaction is automatic in procedures
    
    -- Debit sender
    UPDATE accounts SET balance = balance - amount 
    WHERE user_id = sender_id;
    
    -- Credit receiver
    UPDATE accounts SET balance = balance + amount 
    WHERE user_id = receiver_id;
    
    -- Log transaction
    INSERT INTO transactions (sender_id, receiver_id, amount, created_at)
    VALUES (sender_id, receiver_id, amount, NOW());
    
    COMMIT;  -- Explicit commit
EXCEPTION
    WHEN OTHERS THEN
        ROLLBACK;
        RAISE;
END;
$$ LANGUAGE plpgsql;

-- Call procedure
CALL transfer_funds(1, 2, 100.00);
```

### Triggers
```sql
-- Trigger function
CREATE OR REPLACE FUNCTION update_modified_time()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Create trigger
CREATE TRIGGER trigger_update_modified_time
    BEFORE UPDATE ON users
    FOR EACH ROW
    EXECUTE FUNCTION update_modified_time();

-- Audit trigger
CREATE TABLE audit_log (
    id SERIAL PRIMARY KEY,
    table_name TEXT,
    operation TEXT,
    old_values JSONB,
    new_values JSONB,
    user_name TEXT DEFAULT CURRENT_USER,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE OR REPLACE FUNCTION audit_trigger()
RETURNS TRIGGER AS $$
BEGIN
    INSERT INTO audit_log (table_name, operation, old_values, new_values)
    VALUES (
        TG_TABLE_NAME,
        TG_OP,
        CASE WHEN TG_OP = 'DELETE' THEN to_jsonb(OLD) ELSE NULL END,
        CASE WHEN TG_OP != 'DELETE' THEN to_jsonb(NEW) ELSE NULL END
    );
    
    RETURN CASE WHEN TG_OP = 'DELETE' THEN OLD ELSE NEW END;
END;
$$ LANGUAGE plpgsql;

-- Apply audit trigger to table
CREATE TRIGGER audit_users
    AFTER INSERT OR UPDATE OR DELETE ON users
    FOR EACH ROW EXECUTE FUNCTION audit_trigger();
```

---

## Backup & Recovery

### pg_dump Backup
```bash
# Full database backup
pg_dump -h localhost -U postgres mydb > mydb_backup.sql

# Compressed backup
pg_dump -h localhost -U postgres -Fc mydb > mydb_backup.dump

# Schema only
pg_dump -h localhost -U postgres -s mydb > schema_only.sql

# Data only
pg_dump -h localhost -U postgres -a mydb > data_only.sql

# Specific tables
pg_dump -h localhost -U postgres -t users -t posts mydb > tables_backup.sql

# All databases
pg_dumpall -h localhost -U postgres > all_databases.sql

# With parallel jobs (faster for large databases)
pg_dump -h localhost -U postgres -Fd -j 4 mydb -f mydb_backup/
```

### Restore Commands
```bash
# Restore SQL backup
psql -h localhost -U postgres -d mydb < mydb_backup.sql

# Restore compressed backup
pg_restore -h localhost -U postgres -d mydb mydb_backup.dump

# Restore with parallel jobs
pg_restore -h localhost -U postgres -d mydb -j 4 mydb_backup/

# Create database and restore
createdb -h localhost -U postgres mydb_restored
pg_restore -h localhost -U postgres -d mydb_restored mydb_backup.dump

# Restore specific tables
pg_restore -h localhost -U postgres -d mydb -t users mydb_backup.dump
```

### Point-in-Time Recovery (PITR)
```bash
# Enable WAL archiving in postgresql.conf
archive_mode = on
archive_command = 'cp %p /path/to/wal/archive/%f'
wal_level = replica

# Base backup
pg_basebackup -h localhost -U postgres -D /path/to/backup/ -Ft -z -P

# Recovery configuration
# Create recovery.conf (PostgreSQL < 12) or recovery.signal (PostgreSQL >= 12)
restore_command = 'cp /path/to/wal/archive/%f %p'
recovery_target_time = '2023-01-01 12:00:00'
```

---

## Monitoring & Optimization

### Performance Monitoring
```sql
-- Current queries
SELECT pid, usename, application_name, state, 
       query_start, now() - query_start as duration, query
FROM pg_stat_activity
WHERE state = 'active'
ORDER BY query_start;

-- Slow queries (requires pg_stat_statements extension)
SELECT query, calls, total_time, mean_time, rows
FROM pg_stat_statements
ORDER BY mean_time DESC
LIMIT 10;

-- Database size
SELECT datname, 
       pg_size_pretty(pg_database_size(datname)) as size
FROM pg_database
ORDER BY pg_database_size(datname) DESC;

-- Table sizes
SELECT schemaname, tablename,
       pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) as total_size,
       pg_size_pretty(pg_relation_size(schemaname||'.'||tablename)) as table_size,
       pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename) - pg_relation_size(schemaname||'.'||tablename)) as index_size
FROM pg_tables
WHERE schemaname = 'public'
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;

-- Index usage
SELECT schemaname, tablename, indexname, idx_scan, idx_tup_read, idx_tup_fetch
FROM pg_stat_user_indexes
ORDER BY idx_scan DESC;

-- Cache hit ratio
SELECT datname, 
       round(100.0 * blks_hit / (blks_hit + blks_read), 2) as cache_hit_ratio
FROM pg_stat_database
WHERE blks_read > 0;
```

### Configuration Tuning
```ini
# postgresql.conf optimization settings

# Memory settings
shared_buffers = 256MB                    # 25% of RAM
work_mem = 4MB                            # Per operation
maintenance_work_mem = 64MB               # For vacuum, create index
effective_cache_size = 1GB                # Available OS cache

# Checkpoint settings
checkpoint_completion_target = 0.9
wal_buffers = 16MB
checkpoint_timeout = 10min

# Query planner
random_page_cost = 1.1                    # For SSD storage
effective_io_concurrency = 200            # For SSD storage

# Connection settings
max_connections = 200
shared_preload_libraries = 'pg_stat_statements'

# Logging
log_statement = 'mod'                     # Log modifications
log_min_duration_statement = 1000         # Log slow queries
log_line_prefix = '%t [%p]: [%l-1] user=%u,db=%d,app=%a,client=%h '
```

---

## Extensions

### Popular Extensions
```sql
-- Enable extensions
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";          -- UUID generation
CREATE EXTENSION IF NOT EXISTS "pg_stat_statements"; -- Query statistics
CREATE EXTENSION IF NOT EXISTS "pg_trgm";            -- Trigram matching
CREATE EXTENSION IF NOT EXISTS "btree_gin";          -- GIN indexes for btree
CREATE EXTENSION IF NOT EXISTS "postgis";            -- Geographic data
CREATE EXTENSION IF NOT EXISTS "hstore";             -- Key-value pairs
CREATE EXTENSION IF NOT EXISTS "pgcrypto";           -- Cryptographic functions

-- List installed extensions
SELECT * FROM pg_extension;

-- Available extensions
SELECT * FROM pg_available_extensions ORDER BY name;
```

### UUID Usage
```sql
-- Using UUID extension
CREATE TABLE sessions (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    user_id INTEGER NOT NULL,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

INSERT INTO sessions (user_id) VALUES (1);
```

### Full-Text Search with pg_trgm
```sql
-- Trigram indexes for fuzzy matching
CREATE EXTENSION pg_trgm;

CREATE INDEX idx_name_trgm ON users USING GIN (name gin_trgm_ops);

-- Fuzzy search
SELECT name, similarity(name, 'john') as sim
FROM users
WHERE name % 'john'  -- Similar to 'john'
ORDER BY similarity(name, 'john') DESC;
```

---

## Integration Patterns

### Connection Pooling
```python
# Python with psycopg2 and connection pooling
import psycopg2
from psycopg2 import pool

# Create connection pool
connection_pool = psycopg2.pool.SimpleConnectionPool(
    minconn=1,
    maxconn=20,
    host='localhost',
    database='myapp',
    user='username',
    password='password'
)

def get_user(user_id):
    conn = connection_pool.getconn()
    try:
        with conn.cursor() as cursor:
            cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))
            return cursor.fetchone()
    finally:
        connection_pool.putconn(conn)
```

### Application Integration
```javascript
// Node.js with pg and async/await
const { Pool } = require('pg');

const pool = new Pool({
    host: 'localhost',
    database: 'myapp',
    user: 'username',
    password: 'password',
    port: 5432,
    max: 20,
    idleTimeoutMillis: 30000,
    connectionTimeoutMillis: 2000,
});

// Simple query
async function getUser(userId) {
    const client = await pool.connect();
    try {
        const result = await client.query('SELECT * FROM users WHERE id = $1', [userId]);
        return result.rows[0];
    } finally {
        client.release();
    }
}

// Transaction
async function transferFunds(fromAccount, toAccount, amount) {
    const client = await pool.connect();
    try {
        await client.query('BEGIN');
        
        await client.query(
            'UPDATE accounts SET balance = balance - $1 WHERE id = $2',
            [amount, fromAccount]
        );
        
        await client.query(
            'UPDATE accounts SET balance = balance + $1 WHERE id = $2',
            [amount, toAccount]
        );
        
        await client.query('COMMIT');
    } catch (error) {
        await client.query('ROLLBACK');
        throw error;
    } finally {
        client.release();
    }
}
```

### Streaming Replication
```sql
-- Master configuration (postgresql.conf)
wal_level = replica
max_wal_senders = 3
wal_keep_segments = 64
hot_standby = on

-- pg_hba.conf entry for replication
host replication replicator 192.168.1.100/32 md5

-- Create replication user
CREATE USER replicator REPLICATION LOGIN PASSWORD 'replication_password';

-- Standby server setup
pg_basebackup -h master_ip -D /var/lib/postgresql/data -U replicator -P -W

-- recovery.conf (standby)
standby_mode = 'on'
primary_conninfo = 'host=master_ip port=5432 user=replicator password=replication_password'
trigger_file = '/var/lib/postgresql/trigger_file'
```

---

## Performance Tuning Tips

1. **Use appropriate data types**: Choose the smallest data type that fits your needs
2. **Create proper indexes**: Index columns used in WHERE, ORDER BY, and JOIN clauses
3. **Use EXPLAIN ANALYZE**: Always analyze query performance before optimization
4. **Optimize queries**: Avoid SELECT *, use LIMIT for large result sets
5. **Use connection pooling**: Reuse database connections to reduce overhead
6. **Configure PostgreSQL properly**: Tune memory and checkpoint settings
7. **Regular maintenance**: Run VACUUM and ANALYZE regularly
8. **Monitor performance**: Use pg_stat_statements to identify slow queries
9. **Use partitioning**: Partition very large tables for better performance
10. **Consider materialized views**: Pre-compute expensive queries

---

## Resources
- [PostgreSQL Official Documentation](https://www.postgresql.org/docs/)
- [PostgreSQL Wiki](https://wiki.postgresql.org/)
- [Postgres Performance Guide](https://wiki.postgresql.org/wiki/Performance_Optimization)
- [pgAdmin](https://www.pgadmin.org/) - PostgreSQL administration tool
- [PgBouncer](https://www.pgbouncer.org/) - Connection pooler
- [PostGIS](https://postgis.net/) - Spatial database extension

---
*Comprehensive PostgreSQL reference for advanced database operations. Contributions welcome!*
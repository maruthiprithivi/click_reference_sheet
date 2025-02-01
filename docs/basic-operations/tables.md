# Table Operations

Essential commands for managing tables in ClickHouse Cloud.

## Table Management

### List Tables

```sql
-- Show all tables in current database
SHOW TABLES;

-- Show tables with detailed information
SELECT
    database,
    name AS table_name,
    engine,
    total_rows,
    formatReadableSize(total_bytes) AS total_size,
    formatReadableSize(memory_usage) AS memory_usage
FROM system.tables
WHERE database = currentDatabase()
ORDER BY total_bytes DESC;
```

### Create Table

```sql
-- Basic table creation
CREATE TABLE IF NOT EXISTS table_name
(
    id UInt32,
    timestamp DateTime,
    event_type String,
    user_id String,
    metadata JSON
)
ENGINE = MergeTree()
ORDER BY (timestamp, id);

-- Create table with sampling
CREATE TABLE IF NOT EXISTS table_name
(
    timestamp DateTime,
    user_id String,
    event_type LowCardinality(String),
    value Float64,
    _sample_factor UInt8
)
ENGINE = MergeTree()
ORDER BY (timestamp, user_id)
SAMPLE BY _sample_factor;
```

### Drop Table

```sql
-- Drop table
DROP TABLE IF EXISTS table_name;

-- Drop table with sync (wait for completion)
DROP TABLE IF EXISTS table_name SYNC;
```

### Alter Table

```sql
-- Add column
ALTER TABLE table_name
    ADD COLUMN IF NOT EXISTS new_column_name String;

-- Modify column type
ALTER TABLE table_name
    MODIFY COLUMN column_name UInt64;

-- Drop column
ALTER TABLE table_name
    DROP COLUMN IF EXISTS column_name;

-- Rename table
RENAME TABLE old_table TO new_table;
```

## Table Information

### Table Structure

```sql
-- Show table structure
DESCRIBE table_name;

-- Show create table statement
SHOW CREATE TABLE table_name;

-- Get detailed column information
SELECT
    name,
    type,
    position,
    default_kind,
    default_expression
FROM system.columns
WHERE table = 'table_name'
ORDER BY position;
```

### Table Statistics

```sql
-- Get table size and row count
SELECT
    table,
    formatReadableSize(sum(bytes)) AS size,
    sum(rows) AS row_count,
    max(modification_time) AS last_modified
FROM system.parts
WHERE active AND table = 'table_name'
GROUP BY table;

-- Get detailed table metrics
SELECT
    partition,
    name AS part_name,
    rows,
    formatReadableSize(bytes) AS part_size,
    modification_time
FROM system.parts
WHERE table = 'table_name'
ORDER BY modification_time DESC;
```

## Table Maintenance

### Optimize Table

```sql
-- Basic optimization
OPTIMIZE TABLE table_name FINAL;

-- Optimize specific partition
OPTIMIZE TABLE table_name PARTITION 'partition_id' FINAL;
```

### Table Parts Management

```sql
-- Check table parts
SELECT
    partition,
    name AS part_name,
    active,
    rows,
    formatReadableSize(bytes) AS part_size
FROM system.parts
WHERE table = 'table_name'
ORDER BY bytes DESC;

-- Check parts merging
SELECT *
FROM system.merges
WHERE table = 'table_name';
```

### Table Replication

```sql
-- Check replication status
SELECT
    database,
    table,
    is_leader,
    is_readonly,
    future_parts,
    parts_to_check
FROM system.replicas
WHERE table = 'table_name';
```

## Best Practices

### Storage Optimization

```sql
-- Find tables with small parts
SELECT
    database,
    table,
    count() AS parts_count,
    formatReadableSize(sum(bytes)) AS total_size
FROM system.parts
WHERE active
GROUP BY database, table
HAVING parts_count > 10
ORDER BY parts_count DESC;
```

### Monitoring

```sql
-- Monitor table operations
SELECT
    event_time,
    query,
    read_rows,
    written_rows,
    result_rows
FROM system.query_log
WHERE query LIKE '%table_name%'
  AND type >= 2
  AND event_date >= today() - 1
ORDER BY event_time DESC;
```

### Performance Analysis

```sql
-- Check table performance
SELECT
    table,
    formatReadableSize(sum(bytes)) AS size,
    count() AS parts,
    sum(rows) AS rows,
    avg(marks) AS avg_marks
FROM system.parts
WHERE active
GROUP BY table
ORDER BY sum(bytes) DESC;
```

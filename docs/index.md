# ClickHouse SQL Reference Sheet

A comprehensive SQL command reference sheet for ClickHouse Cloud field engineers. This guide provides essential SQL commands, best practices, and optimization techniques for working with ClickHouse deployments.

## Quick Reference

Here are some of the most frequently used commands:

```sql
-- Check ClickHouse version and health
SELECT version();
SELECT 1;

-- List databases and tables
SHOW DATABASES;
SHOW TABLES;

-- Monitor system
SELECT *
FROM system.metrics
LIMIT 5;

-- Check current queries
SELECT query_id, user, query
FROM system.processes
ORDER BY elapsed DESC;
```

## Basic Operations

### Connection Management

#### Check Current Connection

```sql
SELECT
    currentUser() AS current_user,
    currentDatabase() AS current_database,
    currentQueryId() AS query_id;
```

#### View Active Sessions

```sql
SELECT
    user,
    client_hostname,
    client_name,
    client_version,
    query_start_time,
    query
FROM system.processes
ORDER BY query_start_time DESC;
```

### Database Operations

#### List Databases

```sql
-- Show all databases
SHOW DATABASES;

-- Get detailed information
SELECT
    name,
    engine,
    data_path,
    metadata_path,
    uuid
FROM system.databases;
```

#### Database Management

```sql
-- Create database
CREATE DATABASE IF NOT EXISTS database_name;

-- Drop database
DROP DATABASE IF EXISTS database_name;

-- Switch database
USE database_name;
```

### Table Operations

#### List Tables

```sql
-- Show tables with details
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

#### Create Table

```sql
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
```

#### Alter Table

```sql
-- Add column
ALTER TABLE table_name
    ADD COLUMN IF NOT EXISTS new_column_name String;

-- Modify column
ALTER TABLE table_name
    MODIFY COLUMN column_name UInt64;

-- Drop column
ALTER TABLE table_name
    DROP COLUMN IF EXISTS column_name;
```

## Data Operations

### Insert Commands

#### Basic Insert

```sql
-- Single row
INSERT INTO table_name (column1, column2, column3)
VALUES ('value1', 'value2', 'value3');

-- Multiple rows
INSERT INTO table_name (column1, column2, column3)
VALUES
    ('value1', 'value2', 'value3'),
    ('value4', 'value5', 'value6');
```

#### Insert from SELECT

```sql
INSERT INTO target_table
SELECT *
FROM source_table
WHERE condition;
```

### Query Operations

#### Basic SELECT

```sql
-- Simple query
SELECT *
FROM table_name
LIMIT 10;

-- With filtering and aggregation
SELECT
    column1,
    count() AS count
FROM table_name
WHERE column2 = 'value'
GROUP BY column1
ORDER BY count DESC;
```

#### Advanced Queries

```sql
-- Complex aggregations
SELECT
    toStartOfHour(timestamp) AS hour,
    count() AS events,
    uniqExact(user_id) AS unique_users,
    avg(value) AS avg_value
FROM table_name
GROUP BY hour
ORDER BY hour DESC;

-- Using PREWHERE
SELECT *
FROM table_name
PREWHERE column1 = 'value'
WHERE column2 > 0;
```

### Update and Delete

#### Update Data

```sql
-- Update values
ALTER TABLE table_name
UPDATE column1 = 'new_value'
WHERE column2 = 'condition';

-- Conditional update
ALTER TABLE table_name
UPDATE column1 = CASE
    WHEN column2 > 100 THEN 'high'
    WHEN column2 > 50 THEN 'medium'
    ELSE 'low'
END;
```

#### Delete Data

```sql
-- Delete rows
ALTER TABLE table_name
DELETE WHERE column1 = 'value';

-- Delete old data
ALTER TABLE table_name
DELETE WHERE timestamp < now() - INTERVAL 90 DAY;
```

## System Operations

### Monitoring

#### System Health

```sql
-- Check version and uptime
SELECT
    version() AS version,
    uptime() AS uptime_seconds,
    formatReadableTimeDelta(uptime()) AS uptime_readable;

-- Resource usage
SELECT
    metric,
    value,
    description
FROM system.metrics
WHERE metric LIKE '%Memory%'
ORDER BY metric;
```

#### Query Monitoring

```sql
-- Active queries
SELECT
    query_id,
    user,
    query,
    elapsed,
    formatReadableSize(memory_usage) AS memory
FROM system.processes
WHERE elapsed > 60
ORDER BY elapsed DESC;

-- Query history
SELECT
    type,
    query,
    query_duration_ms,
    read_rows,
    read_bytes,
    memory_usage
FROM system.query_log
WHERE type >= 2
  AND event_date >= today() - 1
ORDER BY query_start_time DESC;
```

### Performance Optimization

#### Query Performance

```sql
-- Analyze slow queries
SELECT
    query,
    query_duration_ms,
    read_rows,
    read_bytes,
    memory_usage
FROM system.query_log
WHERE type >= 2
ORDER BY query_duration_ms DESC
LIMIT 10;

-- Memory usage by query
SELECT
    query_id,
    user,
    formatReadableSize(memory_usage) AS memory,
    elapsed
FROM system.processes
ORDER BY memory_usage DESC;
```

#### Storage Optimization

```sql
-- Table parts analysis
SELECT
    database,
    table,
    count() AS parts_count,
    sum(rows) AS total_rows,
    formatReadableSize(sum(bytes)) AS total_size
FROM system.parts
WHERE active
GROUP BY database, table
HAVING parts_count > 10
ORDER BY parts_count DESC;

-- Compression efficiency
SELECT
    database,
    table,
    formatReadableSize(sum(data_compressed_bytes)) AS compressed,
    formatReadableSize(sum(data_uncompressed_bytes)) AS uncompressed,
    round(sum(data_uncompressed_bytes) / sum(data_compressed_bytes), 2) AS ratio
FROM system.parts
WHERE active
GROUP BY database, table
ORDER BY ratio DESC;
```

## Best Practices

### Query Optimization

1. Always use `LIMIT` when querying large tables
2. Use `PREWHERE` for more efficient filtering
3. Leverage materialized views for common queries
4. Consider sampling for approximate results
5. Monitor and analyze slow queries regularly

### Data Management

1. Use bulk inserts instead of single-row inserts
2. Implement proper partitioning strategies
3. Regular monitoring of table sizes and parts
4. Schedule maintenance operations during off-peak hours
5. Keep track of mutation progress and performance

### System Maintenance

1. Monitor system resources regularly
2. Track query performance and patterns
3. Optimize storage and compression settings
4. Maintain proper backup strategies
5. Regular cleanup of old data

## Troubleshooting

### Common Issues

1. Memory Usage

```sql
-- Check memory-intensive queries
SELECT
    query,
    peak_memory_usage,
    memory_usage,
    query_duration_ms
FROM system.query_log
WHERE event_date >= today() - 1
ORDER BY peak_memory_usage DESC;
```

2. Slow Queries

```sql
-- Find problematic queries
SELECT
    query_id,
    user,
    query,
    elapsed
FROM system.processes
WHERE elapsed > 60;
```

3. System Errors

```sql
-- Check error log
SELECT
    time,
    level,
    message
FROM system.text_log
WHERE level >= 'Error'
ORDER BY time DESC;
```

### Performance Issues

1. Check system metrics
2. Analyze query patterns
3. Review table structure and indexes
4. Monitor background operations
5. Verify resource utilization

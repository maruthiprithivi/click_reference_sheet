# ClickHouse SQL Reference Sheet

A comprehensive SQL command reference sheet for ClickHouse Cloud field engineers. This guide provides essential SQL
commands, best practices, and optimization techniques for working with ClickHouse deployments.

## 📊 Quick Health Checks

Essential commands for quick system health verification:

```sql
-- Basic health check queries
SELECT version();                    -- Check ClickHouse version
SELECT 1;                           -- Simple connectivity test
SELECT currentDatabase();           -- Current database context
SELECT currentUser();               -- Current user context

-- Quick system metrics overview
SELECT metric, value, description
FROM system.metrics
LIMIT 5;                           -- Preview key system metrics
```

## 🔍 Query Analysis and Performance

### Query Execution Analysis

```sql
-- Analyze query execution details including granules and parts
SELECT
    query_id,
    user,
    query,
    read_rows,
    read_bytes,
    result_rows,
    result_bytes,
    memory_usage,
    -- Query execution details
    formatReadableSize(memory_usage) AS memory_used,
    formatReadableQuantity(read_rows) AS rows_read,
    formatReadableSize(read_bytes) AS bytes_read,
    -- Parts and granules information
    read_backup_pieces_marks AS granules_read,      -- Number of granules processed
    marks_bytes AS granules_bytes,                  -- Bytes of mark (granule) data read
    profileEvents['SelectedMarks'] AS marks_selected -- Total marks selected for reading
FROM system.query_log
WHERE type = 'QueryFinish'                         -- Only completed queries
  AND event_date >= today() - 1                    -- Last 24 hours
  AND query NOT LIKE '%system%'                    -- Exclude system queries
ORDER BY memory_usage DESC
LIMIT 10;

-- Detailed granule analysis for specific queries
SELECT
    query_id,
    event_time,
    query,
    -- Granule efficiency metrics
    read_rows / profileEvents['SelectedMarks'] AS rows_per_granule,
    formatReadableSize(read_bytes / profileEvents['SelectedMarks']) AS bytes_per_granule,
    -- Part reading efficiency
    profileEvents['SelectedParts'] AS parts_selected,
    profileEvents['SelectedRanges'] AS ranges_selected,
    -- Memory efficiency
    formatReadableSize(peak_memory_usage) AS peak_memory,
    formatReadableSize(memory_usage) AS avg_memory
FROM system.query_log
WHERE type = 'QueryFinish'
  AND query LIKE '%your_table_name%'              -- Filter for specific table
  AND event_date >= today() - 1
ORDER BY event_time DESC
LIMIT 10;
```

### Query Performance Patterns

```sql
-- Identify slow queries and their patterns
SELECT
    user,
    client_hostname,
    count() AS query_count,
    avg(query_duration_ms) AS avg_duration_ms,
    max(query_duration_ms) AS max_duration_ms,
    sum(read_rows) AS total_rows_processed,
    sum(memory_usage) AS total_memory_used,
    -- Query pattern analysis
    arrayJoin(extractAll(query, '^\\s*(SELECT|INSERT|CREATE|ALTER|DROP)')) AS query_type,
    -- Resource usage patterns
    avg(read_bytes) AS avg_bytes_per_query,
    max(read_bytes) AS max_bytes_per_query
FROM system.query_log
WHERE type = 'QueryFinish'
  AND event_date >= today() - 7                    -- Last week's data
GROUP BY user, client_hostname, query_type
HAVING avg_duration_ms > 1000                      -- Queries averaging over 1 second
ORDER BY avg_duration_ms DESC;

-- Analyze query execution stages
SELECT
    query_id,
    event_time,
    query_kind,
    exception,
    stack_trace,
    -- Timing analysis
    query_start_time,
    query_duration_ms,
    -- Resource usage breakdown
    read_rows,
    read_bytes,
    written_rows,
    written_bytes,
    result_rows,
    result_bytes,
    -- Memory usage analysis
    memory_usage,
    peak_memory_usage
FROM system.query_log
WHERE type IN ('QueryStart', 'QueryFinish', 'ExceptionWhileProcessing')
  AND event_date >= today() - 1
ORDER BY query_id, event_time;
```

## 📈 System Performance Monitoring

### Memory Usage Analysis

```sql
-- Detailed memory usage by query type
SELECT
    user,
    query_kind,
    count() AS query_count,
    -- Memory usage statistics
    formatReadableSize(avg(memory_usage)) AS avg_memory,
    formatReadableSize(max(memory_usage)) AS max_memory,
    formatReadableSize(sum(memory_usage)) AS total_memory,
    -- Memory efficiency metrics
    round(avg(memory_usage / nullIf(read_rows, 0)), 2) AS bytes_per_row,
    round(avg(peak_memory_usage / nullIf(memory_usage, 0)), 2) AS memory_usage_ratio
FROM system.query_log
WHERE type = 'QueryFinish'
  AND event_date >= today() - 1
GROUP BY user, query_kind
ORDER BY total_memory DESC
LIMIT 20;

-- Memory usage trends
SELECT
    toStartOfHour(event_time) AS hour,
    -- Memory usage over time
    formatReadableSize(avg(memory_usage)) AS avg_memory_per_query,
    formatReadableSize(max(memory_usage)) AS max_memory_per_query,
    -- Query patterns
    count() AS query_count,
    sum(read_rows) AS total_rows_processed
FROM system.query_log
WHERE type = 'QueryFinish'
  AND event_date >= today() - 1
GROUP BY hour
ORDER BY hour DESC;
```

### Storage and Compression Analysis

```sql
-- Table storage efficiency analysis
SELECT
    database,
    table,
    -- Size metrics
    formatReadableSize(sum(data_compressed_bytes)) AS compressed_size,
    formatReadableSize(sum(data_uncompressed_bytes)) AS uncompressed_size,
    -- Compression analysis
    round(sum(data_uncompressed_bytes) / sum(data_compressed_bytes), 2) AS compression_ratio,
    -- Part analysis
    count() AS part_count,
    sum(rows) AS total_rows,
    -- Storage efficiency
    round(sum(data_compressed_bytes) / sum(rows), 2) AS bytes_per_row_compressed,
    round(sum(data_uncompressed_bytes) / sum(rows), 2) AS bytes_per_row_uncompressed
FROM system.parts
WHERE active
GROUP BY database, table
ORDER BY compression_ratio DESC;

-- Detailed part analysis
SELECT
    database,
    table,
    partition_id,
    -- Part metrics
    formatReadableSize(data_compressed_bytes) AS compressed_size,
    formatReadableSize(data_uncompressed_bytes) AS uncompressed_size,
    compression_ratio,
    rows,
    -- Mark (granule) statistics
    marks_size,
    marks_bytes,
    -- Modification tracking
    modification_time,
    remove_time
FROM system.parts
WHERE active
  AND database = 'your_database'    -- Replace with your database
ORDER BY data_compressed_bytes DESC
LIMIT 20;
```

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

## Advanced Query Monitoring and Performance Analysis

### 1. Active Query Monitoring

#### Find Currently Running Queries

```sql
-- Active queries with detailed performance metrics
SELECT
    query_id,
    user,
    query,
    elapsed,
    formatReadableSize(memory_usage) AS memory_used,
    read_rows,
    formatReadableSize(read_bytes) AS data_read
FROM system.processes
ORDER BY elapsed DESC;

-- Long-running queries (over 1 minute)
SELECT
    query_id,
    user,
    query,
    elapsed,
    formatReadableSize(memory_usage) AS memory_used
FROM system.processes
WHERE elapsed > 60
ORDER BY elapsed DESC;
```

### 2. Query Performance History

#### Analyze Slow Queries

```sql
-- Slow query log (last 24 hours)
SELECT
    event_time,
    query,
    query_duration_ms,
    read_rows,
    formatReadableSize(read_bytes) AS data_read,
    formatReadableSize(memory_usage) AS peak_memory
FROM system.query_log
WHERE type = 2  -- Query
  AND event_date >= today() - 1
ORDER BY query_duration_ms DESC
LIMIT 20;

-- Query performance by user
SELECT
    user,
    count() AS total_queries,
    avg(query_duration_ms) AS avg_query_time,
    max(query_duration_ms) AS max_query_time
FROM system.query_log
WHERE type = 2
GROUP BY user
ORDER BY total_queries DESC;
```

### 3. Resource Consumption Analysis

#### Memory and CPU Metrics

```sql
-- Memory usage by query
SELECT
    user,
    query_id,
    query,
    formatReadableSize(memory_usage) AS peak_memory,
    formatReadableSize(peak_memory_usage) AS total_peak_memory
FROM system.query_log
WHERE type = 2
ORDER BY peak_memory_usage DESC
LIMIT 20;

-- CPU and thread usage
SELECT
    metric,
    value,
    description
FROM system.metrics
WHERE metric LIKE '%CPU%' OR metric LIKE '%Thread%';

-- Disk and IO metrics
SELECT
    metric,
    value,
    description
FROM system.metrics
WHERE metric LIKE '%Disk%' OR metric LIKE '%IO%';
```

### 4. User Management and RBAC

#### User and Role Management

```sql
-- List all users
SELECT
    name,
    id,
    storage,
    auth_type
FROM system.users;

-- List roles
SELECT
    name,
    id
FROM system.roles;

-- User privileges
SELECT
    user_name,
    role_name,
    privileges
FROM system.role_grants;

-- Detailed user privileges
SELECT
    user_name,
    database,
    table,
    column,
    grant_type
FROM system.grants
ORDER BY user_name, database, table;
```

#### RBAC Examples

```sql
-- Create a role
CREATE ROLE analyst;

-- Grant select privileges
GRANT SELECT
ON database.table
TO analyst;

-- Create a user and assign role
CREATE USER john
IDENTIFIED WITH sha256_password BY 'secure_password'
DEFAULT ROLE analyst;

-- Row-level security
CREATE ROW POLICY limited_view
ON database.table
FOR SELECT
USING department = currentUser();
```

### 5. System and Database Insights

#### Cluster and Database Information

```sql
-- Cluster details
SELECT
    cluster,
    shard_num,
    replica_num,
    host_name,
    port
FROM system.clusters;

-- Database sizes
SELECT
    database,
    formatReadableSize(sum(bytes)) AS total_size,
    count() AS total_tables
FROM system.tables
GROUP BY database
ORDER BY total_size DESC;

-- Table sizes
SELECT
    database,
    table,
    formatReadableSize(sum(bytes)) AS total_size,
    sum(rows) AS total_rows,
    count() AS parts_count
FROM system.parts
WHERE active
GROUP BY database, table
ORDER BY total_size DESC;
```

### 6. Advanced Querying and Indexing

#### Creating and Analyzing Indexes

```sql
-- Create a data skipping index
ALTER TABLE my_table
ADD INDEX idx_name expression TYPE minmax GRANULARITY 3;

-- Analyze index usage
SELECT
    database,
    table,
    name,
    type,
    granularity
FROM system.data_skipping_indices;

-- Full-text search with trigram index
CREATE TABLE my_table (
    text String,
    INDEX text_trigram text TYPE tokenbf_v1(256, 3, 0)
) ENGINE = MergeTree();
```

### 7. Query Settings and Configuration

#### Monitoring Session Settings

```sql
-- View current session settings
SELECT
    name,
    value
FROM system.settings
WHERE changed;

-- Database-level settings
SELECT
    database,
    engine,
    metadata_path
FROM system.databases;

-- Table-level settings
SELECT
    database,
    table,
    engine,
    total_rows,
    total_bytes
FROM system.tables;
```

### 8. Specific Query Monitoring

#### Tracking Queries by User or Table

```sql
-- Queries by specific user
SELECT
    query,
    query_duration_ms,
    read_rows,
    formatReadableSize(read_bytes) AS data_read
FROM system.query_log
WHERE user = 'john'
  AND type = 2
ORDER BY query_duration_ms DESC;

-- Queries on a specific table
SELECT
    user,
    query,
    query_duration_ms
FROM system.query_log
WHERE
    lower(query) LIKE '%mytable%'
    AND type = 2
ORDER BY query_duration_ms DESC;
```

## Best Practices

1. Always use `LIMIT` with large tables
2. Use `PREWHERE` for more efficient filtering
3. Monitor memory usage with settings like `max_memory_usage`
4. Use data skipping indexes for large tables
5. Implement row-level security for sensitive data
6. Regularly review and optimize query performance

## References

- [ClickHouse SQL Reference](https://clickhouse.com/docs/en/sql-reference/statements/select)
- [ClickHouse System Tables](https://clickhouse.com/docs/en/operations/system-tables)
- [ClickHouse RBAC](https://clickhouse.com/docs/en/operations/access-rights)

### Query Optimization Analysis

```sql
-- Analyze query plan and execution
EXPLAIN PLAN
SELECT * FROM your_table WHERE column = 'value';

-- Analyze query pipeline
EXPLAIN PIPELINE
SELECT * FROM your_table WHERE column = 'value';

-- Analyze query AST (Abstract Syntax Tree)
EXPLAIN AST
SELECT * FROM your_table WHERE column = 'value';

-- Analyze query syntax
EXPLAIN SYNTAX
SELECT * FROM your_table WHERE column = 'value';
```

### Index Usage Analysis

```sql
-- Analyze index usage for a table
SELECT
    database,
    table,
    name AS index_name,
    type AS index_type,
    -- Index details
    expr AS index_expression,
    granularity,
    -- Storage metrics
    formatReadableSize(bytes_allocated) AS allocated_size,
    formatReadableSize(data_compressed_bytes) AS compressed_size,
    formatReadableSize(data_uncompressed_bytes) AS uncompressed_size,
    -- Efficiency metrics
    round(data_uncompressed_bytes / nullIf(data_compressed_bytes, 0), 2) AS compression_ratio
FROM system.data_skipping_indices
WHERE database = 'your_database'      -- Replace with your database
  AND table = 'your_table'           -- Replace with your table
ORDER BY bytes_allocated DESC;

-- Monitor index maintenance operations
SELECT
    event_time,
    database,
    table,
    index_name,
    -- Operation details
    operation_type,
    duration_ms,
    -- Resource usage
    read_rows,
    read_bytes,
    written_rows,
    written_bytes
FROM system.part_log
WHERE event_type = 'DataSkippingIndexCreation'
  AND event_date >= today() - 7      -- Last week's operations
ORDER BY event_time DESC;
```

### Query Resource Consumption

```sql
-- Analyze CPU usage by query
SELECT
    query_id,
    user,
    query,
    -- CPU metrics
    ProfileEvents['OSCPUVirtualTimeMicroseconds'] AS cpu_time_us,
    ProfileEvents['OSIOWaitMicroseconds'] AS io_wait_us,
    -- Thread metrics
    ProfileEvents['ThreadsNew'] AS threads_created,
    -- Context switches
    ProfileEvents['ContextSwitchVoluntary'] AS voluntary_switches,
    ProfileEvents['ContextSwitchInvoluntary'] AS involuntary_switches
FROM system.query_log
WHERE type = 'QueryFinish'
  AND event_date >= today() - 1
ORDER BY cpu_time_us DESC
LIMIT 20;

-- Memory allocation analysis
SELECT
    query_id,
    user,
    query,
    -- Memory metrics
    formatReadableSize(memory_usage) AS current_memory,
    formatReadableSize(peak_memory_usage) AS peak_memory,
    -- Memory allocation events
    ProfileEvents['MemoryTracking'] AS memory_tracking,
    ProfileEvents['MemoryTrackingInFunctions'] AS memory_in_functions
FROM system.query_log
WHERE type = 'QueryFinish'
  AND event_date >= today() - 1
ORDER BY peak_memory_usage DESC
LIMIT 20;
```

### Table Access Patterns

```sql
-- Analyze table read patterns
SELECT
    database,
    table,
    -- Access metrics
    count() AS access_count,
    sum(read_rows) AS total_rows_read,
    sum(read_bytes) AS total_bytes_read,
    -- Efficiency metrics
    avg(read_rows) AS avg_rows_per_query,
    formatReadableSize(avg(read_bytes)) AS avg_bytes_per_query,
    -- Time metrics
    avg(query_duration_ms) AS avg_duration_ms
FROM system.query_log
WHERE type = 'QueryFinish'
  AND event_date >= today() - 7
GROUP BY database, table
HAVING access_count > 10             -- Tables accessed more than 10 times
ORDER BY total_bytes_read DESC;

-- Table modification patterns
SELECT
    database,
    table,
    -- Write metrics
    count() AS write_count,
    sum(written_rows) AS total_rows_written,
    formatReadableSize(sum(written_bytes)) AS total_bytes_written,
    -- Time analysis
    avg(query_duration_ms) AS avg_write_duration_ms,
    max(query_duration_ms) AS max_write_duration_ms
FROM system.query_log
WHERE type = 'QueryFinish'
  AND query LIKE '%INSERT%'
  AND event_date >= today() - 7
GROUP BY database, table
ORDER BY total_rows_written DESC;
```

### System Health Monitoring

```sql
-- Monitor system metrics trends
SELECT
    metric,
    value,
    description,
    -- Metric details
    type,
    level,
    -- Add time context
    now() AS current_time
FROM system.metrics
WHERE metric LIKE '%Memory%'         -- Filter for memory metrics
   OR metric LIKE '%CPU%'           -- Filter for CPU metrics
   OR metric LIKE '%Query%'         -- Filter for query metrics
ORDER BY metric;

-- Monitor background processes
SELECT
    type,
    name,
    -- Process details
    elapsed,
    is_cancelled,
    -- Progress tracking
    total_parts_to_do,
    parts_to_do,
    -- Resource usage
    memory_usage,
    cpu_time_microseconds
FROM system.merges
WHERE is_mutation
ORDER BY elapsed DESC;
```

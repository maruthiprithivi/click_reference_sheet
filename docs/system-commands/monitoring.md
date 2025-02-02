# ClickHouse System Monitoring Guide

A comprehensive guide for monitoring ClickHouse deployments.

## System Health Monitoring

### Memory Usage

```sql
-- Current memory usage by query
SELECT
    query_id,
    user,
    formatReadableSize(memory_usage) as mem_used,
    formatReadableSize(peak_memory_usage) as peak_mem,
    query
FROM system.processes
ORDER BY memory_usage DESC;

-- Historical memory usage patterns
SELECT
    query_id,
    formatReadableSize(memory_usage) as mem_used,
    formatReadableSize(peak_memory_usage) as peak_mem,
    query_duration_ms,
    query
FROM system.query_log
WHERE type = 'QueryFinish'
  AND event_time >= now() - INTERVAL 1 DAY
ORDER BY memory_usage DESC
LIMIT 10;
```

### CPU Usage

```sql
-- Current CPU usage by query
SELECT
    query_id,
    user,
    elapsed,
    read_rows,
    formatReadableSize(read_bytes) as read,
    total_rows_approx,
    query
FROM system.processes
ORDER BY elapsed DESC;

-- CPU usage patterns
SELECT
    query,
    count() as execution_count,
    avg(query_duration_ms) as avg_duration_ms,
    max(query_duration_ms) as max_duration_ms,
    sum(read_rows) as total_rows_read,
    sum(result_rows) as total_rows_returned
FROM system.query_log
WHERE type = 'QueryFinish'
  AND event_time >= now() - INTERVAL 1 DAY
GROUP BY query
ORDER BY avg_duration_ms DESC
LIMIT 10;
```

### Disk Usage

```sql
-- Database sizes
SELECT
    name as database,
    formatReadableSize(sum(bytes_on_disk)) as disk_usage,
    sum(rows) as total_rows,
    count() as total_tables
FROM system.tables
GROUP BY database
ORDER BY sum(bytes_on_disk) DESC;

-- Table sizes
SELECT
    database,
    table,
    formatReadableSize(bytes_on_disk) as disk_usage,
    rows as total_rows,
    formatReadableSize(bytes_on_disk / rows) as avg_row_size
FROM system.tables
WHERE database NOT IN ('system')
ORDER BY bytes_on_disk DESC
LIMIT 10;

-- Part sizes and distribution
SELECT
    database,
    table,
    partition,
    formatReadableSize(sum(bytes_on_disk)) as part_size,
    count() as part_count,
    sum(rows) as rows
FROM system.parts
WHERE active
GROUP BY database, table, partition
ORDER BY sum(bytes_on_disk) DESC
LIMIT 10;
```

## Query Performance Monitoring

### Slow Queries

```sql
-- Recent slow queries
SELECT
    query_id,
    user,
    query_duration_ms,
    read_rows,
    formatReadableSize(read_bytes) as data_read,
    result_rows,
    formatReadableSize(memory_usage) as memory,
    query
FROM system.query_log
WHERE type = 'QueryFinish'
  AND query_duration_ms > 1000  -- Queries taking more than 1 second
  AND event_time >= now() - INTERVAL 1 DAY
  AND LOWER(query) NOT LIKE '%system%'  -- Filter out system queries
ORDER BY query_duration_ms DESC
LIMIT 10;

-- Query patterns by duration
SELECT
    user,
    count() as query_count,
    avg(query_duration_ms) as avg_duration_ms,
    max(query_duration_ms) as max_duration_ms
FROM system.query_log
WHERE type = 'QueryFinish'
  AND event_time >= now() - INTERVAL 1 DAY
GROUP BY user
ORDER BY avg_duration_ms DESC;
```

### Query Errors

```sql
-- Recent query errors
SELECT
    user,
    query_id,
    event_time,
    query_duration_ms,
    exception,
    query
FROM system.query_log
WHERE type = 'ExceptionWhileProcessing'
  AND event_time >= now() - INTERVAL 1 DAY
ORDER BY event_time DESC;

-- Error patterns
SELECT
    exception,
    count() as error_count,
    any(query) as sample_query
FROM system.query_log
WHERE type = 'ExceptionWhileProcessing'
  AND event_time >= now() - INTERVAL 1 DAY
GROUP BY exception
ORDER BY error_count DESC;
```

## System Configuration

### Settings Check

```sql
-- Current system settings
SELECT
    name,
    value,
    changed,
    description
FROM system.settings
WHERE changed = 1
ORDER BY name;

-- User settings
SELECT
    *
FROM system.users
FORMAT Vertical;
```

### Replication Status

```sql
-- Replication delays
SELECT
    database,
    table,
    is_leader,
    total_replicas,
    active_replicas,
    formatReadableSize(queue_size) as queue_size,
    absolute_delay
FROM system.replicas
ORDER BY absolute_delay DESC;

-- Replication queue
SELECT
    database,
    table,
    count() as queue_entries,
    max(create_time) as latest_entry
FROM system.replication_queue
GROUP BY database, table
ORDER BY queue_entries DESC;
```

## Best Practices

1. Monitor memory usage regularly to prevent OOM issues
2. Track slow queries and optimize them
3. Keep an eye on disk usage and partition sizes
4. Monitor replication delays in distributed setups
5. Check error patterns periodically
6. Review system settings after updates

## References

- [@ClickHouseSQL System Tables](https://clickhouse.com/docs/en/operations/system-tables)
- [@Web Monitoring](https://clickhouse.com/docs/en/operations/monitoring)
- [@ClickHouseSQL Query Log](https://clickhouse.com/docs/en/operations/system-tables/query_log)

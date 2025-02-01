# System Monitoring

Essential commands for monitoring and diagnosing ClickHouse system performance.

## System Health

### Basic Health Checks

```sql
-- Check system version and uptime
SELECT
    version() AS version,
    uptime() AS uptime_seconds,
    formatReadableTimeDelta(uptime()) AS uptime_readable;

-- Check system settings
SELECT
    name,
    value,
    changed,
    description
FROM system.settings
WHERE changed = 1;
```

### Resource Usage

```sql
-- Memory usage
SELECT
    metric,
    value,
    description
FROM system.metrics
WHERE metric LIKE '%Memory%'
ORDER BY metric;

-- CPU usage
SELECT *
FROM system.metrics
WHERE metric LIKE '%CPU%'
   OR metric LIKE '%Thread%';
```

## Query Monitoring

### Active Queries

```sql
-- List running queries
SELECT
    query_id,
    user,
    address,
    query,
    read_rows,
    read_bytes,
    total_rows_approx,
    formatReadableSize(memory_usage) AS memory,
    elapsed
FROM system.processes
ORDER BY elapsed DESC;

-- Find long-running queries
SELECT
    query_id,
    user,
    query,
    elapsed,
    formatReadableSize(memory_usage) AS memory
FROM system.processes
WHERE elapsed > 60
ORDER BY elapsed DESC;
```

### Query History

```sql
-- Recent query statistics
SELECT
    type,
    query,
    query_start_time,
    query_duration_ms,
    read_rows,
    read_bytes,
    result_rows,
    result_bytes,
    memory_usage
FROM system.query_log
WHERE type >= 2
  AND event_date >= today() - 1
ORDER BY query_start_time DESC
LIMIT 10;

-- Failed queries
SELECT
    event_time,
    query,
    exception,
    stack_trace
FROM system.query_log
WHERE event_date >= today() - 1
  AND exception != ''
ORDER BY event_time DESC;
```

## Storage Monitoring

### Disk Usage

```sql
-- Overall disk usage
SELECT
    name,
    path,
    formatReadableSize(free_space) AS free,
    formatReadableSize(total_space) AS total,
    formatReadableSize(keep_free_space) AS reserved
FROM system.disks;

-- Database sizes
SELECT
    database,
    formatReadableSize(sum(bytes_on_disk)) AS disk_size,
    count() AS total_tables,
    sum(rows) AS total_rows
FROM system.parts
WHERE active
GROUP BY database
ORDER BY sum(bytes_on_disk) DESC;
```

### Table Storage

```sql
-- Table sizes and row counts
SELECT
    database,
    table,
    formatReadableSize(sum(bytes)) AS size,
    sum(rows) AS row_count,
    max(modification_time) AS last_modified,
    count() AS part_count
FROM system.parts
WHERE active
GROUP BY database, table
ORDER BY sum(bytes) DESC;

-- Storage by part
SELECT
    database,
    table,
    partition,
    name AS part_name,
    formatReadableSize(bytes) AS part_size,
    rows,
    modification_time
FROM system.parts
WHERE active
ORDER BY bytes DESC
LIMIT 10;
```

## Performance Monitoring

### System Metrics

```sql
-- Key performance metrics
SELECT *
FROM system.metrics
WHERE metric IN (
    'Query',
    'QueryThread',
    'QueryPreempted',
    'TCPConnection',
    'HTTPConnection',
    'InterserverConnection',
    'ReadbackgroundPoolTask'
);

-- Asynchronous metrics
SELECT *
FROM system.asynchronous_metrics
WHERE metric LIKE '%RWLock%'
   OR metric LIKE '%Thread%'
   OR metric LIKE '%Memory%';
```

### Background Operations

```sql
-- Check merge operations
SELECT *
FROM system.merges
WHERE is_mutation = 0;

-- Monitor background tasks
SELECT *
FROM system.background_processing_pool;
```

## Replication Status

### Replica Health

```sql
-- Check replica status
SELECT
    database,
    table,
    is_leader,
    is_readonly,
    absolute_delay,
    queue_size,
    inserts_in_queue,
    merges_in_queue
FROM system.replicas
ORDER BY absolute_delay DESC;

-- Find replication issues
SELECT
    database,
    table,
    total_replicas,
    active_replicas,
    is_readonly,
    is_session_expired,
    future_parts,
    parts_to_check
FROM system.replicas
WHERE is_readonly
   OR is_session_expired
   OR parts_to_check > 0;
```

## System Diagnostics

### Error Monitoring

```sql
-- Check system errors
SELECT
    time,
    level,
    message
FROM system.text_log
WHERE level >= 'Error'
  AND event_date >= today() - 1
ORDER BY time DESC;

-- Monitor warnings
SELECT
    time,
    level,
    message
FROM system.text_log
WHERE level = 'Warning'
  AND event_date >= today() - 1
ORDER BY time DESC;
```

### Configuration Checks

```sql
-- Check important settings
SELECT
    name,
    value,
    changed,
    description
FROM system.settings
WHERE name IN (
    'max_memory_usage',
    'max_concurrent_queries',
    'max_connections',
    'max_table_size_to_drop',
    'max_partition_size_to_drop'
);

-- Review access rights
SELECT
    user,
    access_type,
    database,
    table,
    column,
    is_partial_revoke
FROM system.access_rights
ORDER BY user, access_type;
```

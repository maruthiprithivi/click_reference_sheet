# Performance Optimization

Essential commands and techniques for optimizing ClickHouse performance.

## Query Performance

### Query Analysis

```sql
-- Analyze query performance
SELECT
    query,
    query_duration_ms,
    read_rows,
    read_bytes,
    written_rows,
    written_bytes,
    result_rows,
    result_bytes,
    memory_usage
FROM system.query_log
WHERE type >= 2
  AND event_date >= today() - 1
ORDER BY query_duration_ms DESC
LIMIT 10;

-- Query profiling
SELECT
    ProfileEvents.Names AS metric,
    ProfileEvents.Values AS value
FROM system.query_log
ARRAY JOIN ProfileEvents
WHERE query_id = 'your_query_id'
  AND type = 'QueryFinish';
```

### Slow Queries

```sql
-- Find slow queries
SELECT
    query_id,
    user,
    query,
    read_rows,
    read_bytes,
    elapsed,
    memory_usage
FROM system.processes
WHERE elapsed > 10
ORDER BY elapsed DESC;

-- Analyze query patterns
SELECT
    user,
    count() AS query_count,
    avg(query_duration_ms) AS avg_duration,
    max(query_duration_ms) AS max_duration,
    sum(memory_usage) AS total_memory
FROM system.query_log
WHERE type >= 2
  AND event_date >= today() - 1
GROUP BY user
ORDER BY avg_duration DESC;
```

## Memory Management

### Memory Usage

```sql
-- Current memory usage
SELECT
    metric,
    value,
    description
FROM system.metrics
WHERE metric LIKE '%Memory%'
ORDER BY metric;

-- Memory usage by query
SELECT
    query_id,
    user,
    query,
    formatReadableSize(memory_usage) AS memory,
    elapsed
FROM system.processes
ORDER BY memory_usage DESC;
```

### Memory Settings

```sql
-- Check memory-related settings
SELECT
    name,
    value,
    changed,
    description
FROM system.settings
WHERE name LIKE '%memory%'
   OR name LIKE '%buffer%';

-- Monitor memory limits
SELECT *
FROM system.metrics
WHERE metric IN (
    'MemoryTracking',
    'MemoryTrackingInBackgroundProcessingPool',
    'MemoryTrackingInBackgroundMoveProcessingPool',
    'MemoryTrackingInBackgroundSchedulePool'
);
```

## Storage Optimization

### Table Optimization

```sql
-- Find tables with many parts
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

-- Check compression ratio
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

### Part Management

```sql
-- Monitor part merges
SELECT
    database,
    table,
    partition_id,
    elapsed,
    progress,
    num_parts,
    result_part_name,
    total_size_bytes_compressed,
    is_mutation
FROM system.merges
ORDER BY elapsed DESC;

-- Check part distribution
SELECT
    database,
    table,
    partition,
    count() AS part_count,
    sum(rows) AS total_rows,
    formatReadableSize(sum(bytes)) AS total_size
FROM system.parts
WHERE active
GROUP BY database, table, partition
ORDER BY part_count DESC;
```

## System Optimization

### Background Tasks

```sql
-- Monitor background operations
SELECT *
FROM system.background_processing_pool
ORDER BY elapsed DESC;

-- Check background metrics
SELECT *
FROM system.metrics
WHERE metric LIKE '%Background%'
ORDER BY metric;
```

### Resource Usage

```sql
-- CPU usage metrics
SELECT *
FROM system.metrics
WHERE metric LIKE '%CPU%'
   OR metric LIKE '%Thread%';

-- IO metrics
SELECT *
FROM system.metrics
WHERE metric LIKE '%IO%'
   OR metric LIKE '%Disk%';
```

## Performance Tuning

### Settings Optimization

```sql
-- Important performance settings
SELECT
    name,
    value,
    changed,
    description
FROM system.settings
WHERE name IN (
    'max_threads',
    'max_insert_threads',
    'max_compress_block_size',
    'min_compress_block_size',
    'max_query_size',
    'max_memory_usage',
    'max_memory_usage_for_user'
);

-- Check current optimization settings
SELECT
    name,
    value,
    changed,
    description
FROM system.settings
WHERE name LIKE '%optimize%'
   OR name LIKE '%parallel%';
```

### Query Cache

```sql
-- Monitor query cache
SELECT *
FROM system.metrics
WHERE metric LIKE '%Cache%';

-- Check cache effectiveness
SELECT
    metric,
    value
FROM system.metrics
WHERE metric IN (
    'MarkCacheBytes',
    'MarkCacheFiles',
    'UncompressedCacheBytes',
    'UncompressedCacheCells'
);
```

## Performance Monitoring

### Real-time Metrics

```sql
-- Current performance metrics
SELECT
    metric,
    value,
    description
FROM system.metrics
WHERE metric IN (
    'Query',
    'QueryThread',
    'QueryPreempted',
    'QueryMemoryTracking',
    'CompressedReadBufferBlocks',
    'CompressedReadBufferBytes'
);

-- System loads
SELECT *
FROM system.asynchronous_metrics
WHERE metric LIKE '%Load%'
   OR metric LIKE '%CPU%'
   OR metric LIKE '%Memory%';
```

### Performance Trends

```sql
-- Query performance over time
SELECT
    toStartOfHour(event_time) AS hour,
    count() AS query_count,
    avg(query_duration_ms) AS avg_duration,
    max(query_duration_ms) AS max_duration,
    sum(memory_usage) AS total_memory
FROM system.query_log
WHERE type >= 2
  AND event_date >= today() - 1
GROUP BY hour
ORDER BY hour DESC;

-- Resource usage trends
SELECT
    toStartOfHour(event_time) AS hour,
    avg(read_rows) AS avg_read_rows,
    avg(read_bytes) AS avg_read_bytes,
    avg(memory_usage) AS avg_memory_usage
FROM system.query_log
WHERE type >= 2
  AND event_date >= today() - 1
GROUP BY hour
ORDER BY hour DESC;
```

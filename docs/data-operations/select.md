# Query Operations

Essential commands and optimization techniques for querying data in ClickHouse.

## Basic Queries

### Simple SELECT

```sql
-- Basic SELECT
SELECT *
FROM table_name
LIMIT 10;

-- Select specific columns
SELECT
    column1,
    column2,
    count() AS count
FROM table_name
WHERE column1 = 'value'
GROUP BY column1, column2
ORDER BY count DESC
LIMIT 100;
```

### Filtering and Sorting

```sql
-- Complex filtering
SELECT *
FROM table_name
WHERE column1 IN ('value1', 'value2')
  AND column2 BETWEEN '2024-01-01' AND '2024-02-01'
  AND column3 LIKE '%pattern%'
ORDER BY column1 DESC
LIMIT 1000;

-- Using PREWHERE for optimization
SELECT *
FROM table_name
PREWHERE column1 = 'value'
WHERE column2 > 0
LIMIT 100;
```

## Advanced Queries

### Aggregations

```sql
-- Basic aggregations
SELECT
    toStartOfHour(timestamp) AS hour,
    count() AS events,
    uniqExact(user_id) AS unique_users,
    avg(value) AS avg_value,
    quantile(0.95)(duration) AS p95_duration
FROM table_name
GROUP BY hour
ORDER BY hour DESC;

-- Complex aggregations
SELECT
    user_id,
    groupArray(event_type) AS events,
    groupArrayMovingAvg(5)(value) AS moving_avg
FROM table_name
GROUP BY user_id
HAVING length(events) > 10;
```

### Joins

```sql
-- Inner JOIN
SELECT
    t1.id,
    t1.timestamp,
    t2.user_name
FROM table1 AS t1
INNER JOIN table2 AS t2
ON t1.user_id = t2.id;

-- LEFT ARRAY JOIN
SELECT
    user_id,
    arrayJoin(events) AS event
FROM (
    SELECT
        user_id,
        groupArray(event_type) AS events
    FROM table_name
    GROUP BY user_id
);
```

## Query Optimization

### Using SAMPLE

```sql
-- Query with sampling
SELECT
    count() * any(sampling_ratio) AS estimated_total,
    avg(value) AS estimated_avg
FROM table_name
SAMPLE 0.1;

-- Sample with offset
SELECT count()
FROM table_name
SAMPLE 0.1 OFFSET 0.5;
```

### Materialized Views

```sql
-- Query materialized view
SELECT *
FROM mv_table_name
WHERE timestamp >= now() - INTERVAL 1 HOUR;

-- Check materialized view status
SELECT
    database,
    table,
    engine,
    total_rows,
    total_bytes
FROM system.tables
WHERE name LIKE 'mv_%';
```

## Performance Analysis

### Query Profiling

```sql
-- Enable profiling
SET query_profiler_real_time_period_ns=10000000;
SET allow_introspection_functions=1;

-- Profile query
SELECT
    ProfileEvents.Names AS metric,
    ProfileEvents.Values AS value
FROM system.query_log
ARRAY JOIN ProfileEvents
WHERE query_id = 'your_query_id'
  AND type = 'QueryFinish';
```

### Query Analysis

```sql
-- Analyze query performance
SELECT
    query,
    read_rows,
    read_bytes,
    written_rows,
    written_bytes,
    memory_usage,
    query_duration_ms
FROM system.query_log
WHERE type >= 2
  AND event_date >= today() - 1
ORDER BY memory_usage DESC
LIMIT 10;
```

## Best Practices

### Query Optimization Tips

```sql
-- Check table statistics
SELECT
    table,
    sum(rows) AS total_rows,
    formatReadableSize(sum(bytes)) AS total_size,
    min(min_date) AS min_date,
    max(max_date) AS max_date
FROM system.parts
WHERE active
GROUP BY table;

-- Analyze column cardinality
SELECT
    column_name,
    uniqExact(column_value) AS cardinality
FROM (
    SELECT
        arrayJoin(['column1', 'column2']) AS column_name,
        column1 AS column_value
    FROM table_name
);
```

### Common Optimizations

1. Use PREWHERE for filtering
2. Leverage materialized views for common queries
3. Use appropriate indexes (ORDER BY columns)
4. Consider sampling for approximate results
5. Use FINAL modifier only when necessary

### Query Monitoring

```sql
-- Monitor long-running queries
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

-- Check query cache usage
SELECT
    metric,
    value
FROM system.metrics
WHERE metric LIKE '%Cache%';
```

## Troubleshooting

### Memory Usage

```sql
-- Check memory-intensive queries
SELECT
    query,
    peak_memory_usage,
    memory_usage,
    query_duration_ms
FROM system.query_log
WHERE event_date >= today() - 1
ORDER BY peak_memory_usage DESC
LIMIT 10;
```

### Query Errors

```sql
-- Find failed queries
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

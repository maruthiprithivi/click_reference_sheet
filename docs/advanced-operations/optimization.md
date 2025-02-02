# Query Optimization Guide

A comprehensive guide to optimizing ClickHouse queries for better performance.

## Overview

This guide covers various techniques for optimizing query performance in ClickHouse, including:

- Query structure optimization
- Index usage
- Memory management
- Data skipping optimization

[@ClickHouseSQL Reference](https://clickhouse.com/docs/en/sql-reference/statements/select)

## Query Structure Optimization

### Using PREWHERE

```sql
-- Example: Using PREWHERE for better filtering performance
SELECT
    user_id,
    timestamp,
    event_type
FROM events
PREWHERE event_date = today()  -- Executed before WHERE, reduces data read
WHERE event_type = 'click'
LIMIT 100;
```

**When to use**: Use PREWHERE when filtering on columns that can significantly reduce the data before processing.

### Optimizing JOINs

```sql
-- Example: Join optimization with proper table order
SELECT
    u.name,
    count(e.event_id) as event_count
FROM events e
INNER JOIN users u ON u.id = e.user_id  -- Smaller table on right
WHERE e.event_date >= today() - 7
GROUP BY u.name;
```

**Best Practice**: Place larger tables first in JOIN operations.

## Index Usage

### Primary Key Optimization

```sql
-- Example: Query using primary key for efficient data access
SELECT *
FROM events
WHERE (event_date, user_id) = ('2024-02-01', 123)  -- Uses primary key
LIMIT 10;

-- Check primary key usage
EXPLAIN indexes = 1
SELECT * FROM events
WHERE event_date = '2024-02-01';
```

### Skip Index Creation and Usage

```sql
-- Create skip index
ALTER TABLE events
ADD INDEX event_type_idx event_type TYPE minmax GRANULARITY 4;

-- Query using skip index
SELECT count()
FROM events
WHERE event_type = 'purchase'  -- Will use skip index
  AND event_date >= today() - 7;
```

## Memory Management

### Memory Settings

```sql
-- Set memory limits for query
SET max_memory_usage = 20000000000;  -- 20GB limit
SET max_bytes_before_external_sort = 10000000000;  -- 10GB for sorting
```

### Using Sampling for Large Tables

```sql
-- Example: Approximate analysis using sampling
SELECT
    event_type,
    count() * 100 as estimated_total
FROM events
SAMPLE 0.01  -- 1% sample
GROUP BY event_type;
```

## Data Skipping Optimization

### Partition Pruning

```sql
-- Example: Query leveraging partitioning
SELECT count()
FROM events
WHERE event_date BETWEEN '2024-01-01' AND '2024-01-31'  -- Uses partition pruning
  AND user_id = 123;

-- Check partition usage
SELECT
    partition_id,
    rows,
    formatReadableSize(bytes_on_disk) as size
FROM system.parts
WHERE table = 'events'
  AND active = 1;
```

## Query Analysis Tools

```sql
-- Analyze query execution plan
EXPLAIN pipeline
SELECT * FROM events WHERE event_date = today();

-- Profile query execution
SET allow_introspection_functions=1;
SELECT * FROM events WHERE event_date = today() SETTINGS profile=1;
```

For performance monitoring queries, see [System Monitoring Guide](../system-commands/monitoring.md).

## Best Practices

1. Always use appropriate indexes
2. Leverage partitioning for large tables
3. Use PREWHERE for early data filtering
4. Monitor and limit memory usage
5. Use sampling for approximate analysis of large datasets
6. Regularly analyze query performance
7. Optimize JOIN operations

## References

- [@ClickHouseSQL Query Optimization](https://clickhouse.com/docs/en/sql-reference/statements/select#optimization)
- [@Web Performance Tuning](https://clickhouse.com/docs/en/operations/performance-tuning)
- [@ClickHouseSQL System Tables](https://clickhouse.com/docs/en/operations/system-tables)

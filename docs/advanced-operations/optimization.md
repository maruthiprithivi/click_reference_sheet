# Query Optimization Guide

A comprehensive guide to optimizing ClickHouse queries for better performance.

## Overview

This guide covers various techniques for optimizing query performance in ClickHouse, including:

- Query structure optimization
- Index usage
- Memory management
- Data skipping optimization

<!-- [@ClickHouseSQL Reference](https://clickhouse.com/docs/en/sql-reference/statements/select) -->

## Query Structure Optimization

### Using PREWHERE

PREWHERE is a powerful optimization feature in ClickHouse designed to improve query performance by filtering data early
in the query execution process. This early filtering reduces the amount of data read from disk, leading to faster query
execution times.

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

-- Example: Multiple columns in PREWHERE
SELECT
    user_id,
    event_type,
    value
FROM events
PREWHERE (event_date = today() AND user_id = 123)  -- Highly selective combined condition
WHERE value > 1000;

-- Example: Using PREWHERE with small columns
SELECT
    description,  -- Large column
    status_code,  -- Small column
    user_id       -- Small column
FROM events
PREWHERE status_code = 200  -- Filter on small column first
WHERE description LIKE '%error%';  -- Filter on large column later
```

#### Best Practices for PREWHERE

1. **Selective Filtering**

   - Use PREWHERE for highly selective conditions (e.g., unique identifiers)
   - Avoid using it for conditions matching large portions of data
   - Combine multiple conditions in PREWHERE when they remain highly selective

2. **Column Size Consideration**

   - Small Columns:
     - Use PREWHERE with columns having minimal storage (e.g., Int8, Int16, Float32)
     - Examples: status codes, age, small fixed-length strings
   - Large Columns:
     - Avoid using PREWHERE with large columns (e.g., long texts, large arrays)
     - Examples: detailed descriptions, JSON blobs, large string columns

3. **Multiple Column Strategy**

   - Combine multiple small columns in PREWHERE for better filtering
   - Ensure combined conditions maintain high selectivity
   - Order conditions from most to least selective

4. **PREWHERE and WHERE Combination**

   - Use PREWHERE for early, highly selective filtering
   - Use WHERE for additional, less selective conditions
   - Let PREWHERE handle small columns while WHERE processes larger ones

#### Additional Considerations

- **Automatic Optimization**: ClickHouse can automatically move WHERE conditions to PREWHERE
- **Manual Control**: Explicitly using PREWHERE provides finer control over optimization
- **Engine Support**: Ensure your table engine (MergeTree family) supports PREWHERE
- **Performance Monitoring**: Regularly analyze query performance with and without PREWHERE

<!-- ### Optimizing JOINs

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

**Best Practice**: Place larger tables first in JOIN operations. -->

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

<!-- ## Best Practices

1. Always use appropriate indexes
2. Leverage partitioning for large tables
3. Use PREWHERE for early data filtering
4. Monitor and limit memory usage
5. Use sampling for approximate analysis of large datasets
6. Regularly analyze query performance
7. Optimize JOIN operations -->

<!--
## References

- [@ClickHouseSQL Query Optimization](https://clickhouse.com/docs/en/sql-reference/statements/select#optimization)
- [@Web Performance Tuning](https://clickhouse.com/docs/en/operations/performance-tuning)
- [@ClickHouseSQL System Tables](https://clickhouse.com/docs/en/operations/system-tables) -->

# Data Insertion

Commands and best practices for inserting data into ClickHouse tables.

## Basic Insert Operations

### Single Row Insert

```sql
-- Basic INSERT
INSERT INTO table_name (column1, column2, column3)
VALUES ('value1', 'value2', 'value3');

-- Insert with DEFAULT values
INSERT INTO table_name
VALUES ('value1', DEFAULT, 'value3');
```

### Bulk Insert

```sql
-- Multiple rows INSERT
INSERT INTO table_name (column1, column2, column3)
VALUES
    ('value1', 'value2', 'value3'),
    ('value4', 'value5', 'value6'),
    ('value7', 'value8', 'value9');
```

### Insert from SELECT

```sql
-- Insert data from another table
INSERT INTO target_table
SELECT *
FROM source_table
WHERE condition;

-- Insert with column transformation
INSERT INTO target_table (column1, column2, transformed_column)
SELECT
    column1,
    column2,
    transform_function(column3)
FROM source_table;
```

## Advanced Insert Operations

### Insert with CAST

```sql
-- Insert with type casting
INSERT INTO table_name
SELECT
    CAST(column1 AS UInt32) AS id,
    CAST(column2 AS DateTime) AS timestamp,
    column3
FROM source_table;
```

### Asynchronous Insert

```sql
-- Enable asynchronous inserts
SET async_insert = 1;
SET wait_for_async_insert = 0;

-- Insert with async settings
INSERT INTO table_name
SELECT * FROM source_table;
```

### Insert with Deduplication

```sql
-- Check current deduplication window
SELECT value
FROM system.settings
WHERE name = 'merge_tree_min_rows_for_concurrent_read';

-- Insert with deduplication
INSERT INTO table_name
SELECT *
FROM source_table
GROUP BY id
HAVING max(timestamp);
```

## Monitoring Insert Operations

### Track Insert Performance

```sql
-- Monitor recent inserts
SELECT
    query_id,
    event_time,
    query,
    written_rows,
    written_bytes,
    peak_memory_usage
FROM system.query_log
WHERE type = 2  -- Type INSERT
  AND event_date >= today() - 1
ORDER BY event_time DESC;
```

### Check Insert Errors

```sql
-- View failed inserts
SELECT
    event_time,
    query,
    exception,
    stack_trace
FROM system.query_log
WHERE type = 2  -- Type INSERT
  AND exception != ''
  AND event_date >= today() - 1
ORDER BY event_time DESC;
```

## Best Practices

### Optimize Insert Performance

```sql
-- Check current insert-related settings
SELECT name, value, description
FROM system.settings
WHERE name LIKE '%insert%'
   OR name LIKE '%buffer%';

-- Monitor buffer table usage
SELECT
    database,
    table,
    blocks_inserted,
    blocks_removed,
    rows
FROM system.buffer_usage;
```

### Batch Insert Recommendations

1. Use bulk inserts instead of single-row inserts
2. Consider using Buffer tables for high-frequency inserts
3. Monitor and adjust buffer settings based on workload
4. Use async inserts for better throughput when appropriate

### Insert with Buffer Table

```sql
-- Create buffer table
CREATE TABLE buffer_table AS source_table
ENGINE = Buffer(database, source_table, 16, 10, 100, 10000, 1000000, 10000000, 100000000);

-- Insert into buffer table
INSERT INTO buffer_table
SELECT * FROM temporary_data;
```

## Troubleshooting

### Common Issues

```sql
-- Check for blocked inserts
SELECT
    query_id,
    user,
    query,
    elapsed
FROM system.processes
WHERE query LIKE 'INSERT%'
ORDER BY elapsed DESC;

-- Monitor insert queues
SELECT *
FROM system.metrics
WHERE metric LIKE '%Insert%'
   OR metric LIKE '%Background%';
```

### Data Quality Checks

```sql
-- Verify inserted data
SELECT
    count() AS row_count,
    min(timestamp) AS earliest_record,
    max(timestamp) AS latest_record,
    uniqExact(id) AS unique_ids
FROM table_name
WHERE timestamp >= today() - 1;

-- Check for data anomalies
SELECT
    toHour(timestamp) AS hour,
    count() AS records_per_hour
FROM table_name
WHERE timestamp >= today() - 1
GROUP BY hour
ORDER BY hour;
```

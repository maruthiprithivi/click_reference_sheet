# Update and Delete Operations

Commands and best practices for updating and deleting data in ClickHouse.

## ALTER TABLE MUTATIONS

### Basic UPDATE

```sql
-- Update specific values
ALTER TABLE table_name
UPDATE column1 = 'new_value'
WHERE column2 = 'condition';

-- Update with expression
ALTER TABLE table_name
UPDATE column1 = column2 * 2
WHERE column3 > 100;
```

### Basic DELETE

```sql
-- Delete specific rows
ALTER TABLE table_name
DELETE WHERE column1 = 'value';

-- Delete with complex condition
ALTER TABLE table_name
DELETE WHERE (column1 = 'value1' AND column2 > 100)
   OR (column3 LIKE '%pattern%');
```

## Monitoring Mutations

### Check Mutation Status

```sql
-- View active mutations
SELECT
    database,
    table,
    mutation_id,
    command,
    create_time,
    is_done
FROM system.mutations
WHERE is_done = 0;

-- View completed mutations
SELECT
    database,
    table,
    mutation_id,
    command,
    create_time,
    is_done,
    parts_to_do_names
FROM system.mutations
WHERE is_done = 1
ORDER BY create_time DESC
LIMIT 10;
```

### Track Mutation Progress

```sql
-- Monitor mutation progress
SELECT
    database,
    table,
    mutation_id,
    create_time,
    parts_to_do,
    is_done,
    latest_failed_part,
    latest_fail_reason
FROM system.mutations
WHERE table = 'table_name'
ORDER BY create_time DESC;
```

## Advanced Operations

### Conditional Updates

```sql
-- Update with CASE statement
ALTER TABLE table_name
UPDATE column1 = CASE
    WHEN column2 > 100 THEN 'high'
    WHEN column2 > 50 THEN 'medium'
    ELSE 'low'
END
WHERE column2 IS NOT NULL;

-- Update multiple columns
ALTER TABLE table_name
UPDATE
    column1 = column1 * 2,
    column2 = 'updated',
    column3 = now()
WHERE column4 IN (1, 2, 3);
```

### Batch Operations

```sql
-- Delete in batches
ALTER TABLE table_name
DELETE WHERE column1 IN (
    SELECT id
    FROM batch_delete_table
    WHERE status = 'to_delete'
);

-- Update in batches
ALTER TABLE table_name
UPDATE column1 = 'updated'
WHERE column2 IN (
    SELECT id
    FROM batch_update_table
    WHERE needs_update = 1
);
```

## Best Practices

### Performance Optimization

```sql
-- Check parts affected by mutation
SELECT
    database,
    table,
    mutation_id,
    parts_to_do,
    parts_to_do_names
FROM system.mutations
WHERE is_done = 0;

-- Monitor system resources during mutation
SELECT
    metric,
    value
FROM system.metrics
WHERE metric LIKE '%Mutation%';
```

### Mutation Management

```sql
-- Kill stuck mutation
KILL MUTATION
WHERE table = 'table_name'
  AND mutation_id = 'mutation_id';

-- Check mutation errors
SELECT
    database,
    table,
    mutation_id,
    latest_fail_reason,
    latest_failed_part
FROM system.mutations
WHERE latest_fail_reason != '';
```

## Data Maintenance

### Regular Cleanup

```sql
-- Delete old data
ALTER TABLE table_name
DELETE WHERE timestamp < now() - INTERVAL 90 DAY;

-- Update expired records
ALTER TABLE table_name
UPDATE status = 'expired'
WHERE expiry_date < now()
  AND status = 'active';
```

### Data Quality

```sql
-- Fix invalid values
ALTER TABLE table_name
UPDATE
    numeric_column = 0
WHERE numeric_column < 0;

-- Standardize data
ALTER TABLE table_name
UPDATE
    string_column = upperUTF8(string_column)
WHERE string_column != upperUTF8(string_column);
```

## Troubleshooting

### Common Issues

```sql
-- Check for blocked mutations
SELECT
    database,
    table,
    mutation_id,
    create_time,
    latest_fail_reason
FROM system.mutations
WHERE is_done = 0
  AND create_time < now() - INTERVAL 1 HOUR;

-- Monitor mutation impact
SELECT
    table,
    sum(rows) AS total_rows,
    formatReadableSize(sum(bytes)) AS total_size
FROM system.parts
WHERE table = 'table_name'
  AND active
GROUP BY table;
```

### Best Practices Summary

1. Always use WHERE clause to limit the scope of mutations
2. Monitor mutation progress and resource usage
3. Consider breaking large mutations into smaller batches
4. Have a rollback plan for large-scale mutations
5. Schedule mutations during off-peak hours
6. Regular monitoring of mutation performance and status

### Mutation Settings

```sql
-- Check mutation-related settings
SELECT
    name,
    value,
    description
FROM system.settings
WHERE name LIKE '%mutation%'
   OR name LIKE '%alter%';

-- Monitor mutation metrics
SELECT *
FROM system.metrics
WHERE metric LIKE '%Mutation%'
   OR metric LIKE '%Alter%';
```

# ClickHouse Maintenance Guide

A comprehensive guide for maintaining ClickHouse deployments.

## Regular Maintenance Tasks

### Data Retention

```sql
-- Check tables with TTL
SELECT
    database,
    table,
    name as ttl_name,
    expression as ttl_expression
FROM system.ttl_expressions
ORDER BY database, table;

-- Apply TTL manually
ALTER TABLE database.table MODIFY TTL event_date + INTERVAL 90 DAY;
```

### Table Optimization

```sql
-- Optimize table storage
OPTIMIZE TABLE database.table FINAL;

-- Check if table needs optimization
SELECT
    database,
    table,
    formatReadableSize(data_uncompressed_bytes) as uncompressed_size,
    formatReadableSize(data_compressed_bytes) as compressed_size,
    rows_count
FROM system.parts_columns
WHERE (database, table) IN (
    SELECT database, table
    FROM system.tables
    WHERE engine = 'MergeTree'
)
GROUP BY database, table
HAVING count() > 10  -- Tables with many parts
ORDER BY uncompressed_size DESC;
```

### Backup Operations

```sql
-- Create backup
BACKUP TABLE database.table TO '/path/to/backup';

-- Restore from backup
RESTORE TABLE database.table FROM '/path/to/backup';

-- Check backup status
SELECT *
FROM system.backup_log
ORDER BY start_time DESC
LIMIT 10;
```

## Maintenance Schedule

### Daily Tasks

1. Check system health (see [Monitoring Guide](../system-commands/monitoring.md))
2. Review error logs
3. Verify backup completion
4. Check replication status

### Weekly Tasks

1. Optimize frequently modified tables
2. Review and adjust TTL settings
3. Clean up old backups
4. Update system settings if needed

### Monthly Tasks

1. Full system audit
2. Capacity planning
3. Performance optimization
4. Security review

## Data Management

### Table Maintenance

```sql
-- Analyze table structure
SHOW CREATE TABLE database.table;

-- Check table dependencies
SELECT
    database,
    table,
    name as dependency_name,
    type as dependency_type
FROM system.table_dependencies
WHERE database = 'your_database'
  AND table = 'your_table';

-- Verify table consistency
CHECK TABLE database.table;
```

### Storage Management

```sql
-- Move table to different disk
ALTER TABLE database.table MOVE PARTITION tuple() TO DISK 'new_disk';

-- Attach/detach partition
ALTER TABLE database.table DETACH PARTITION partition_id;
ALTER TABLE database.table ATTACH PARTITION partition_id;
```

## System Administration

### Configuration Management

```sql
-- Export configuration
SELECT *
FROM system.settings
WHERE changed = 1
FORMAT TSV;

-- Import configuration
SET setting_name = 'value';
```

### User Management

```sql
-- Create new user
CREATE USER username
IDENTIFIED WITH sha256_password BY 'password'
SETTINGS max_memory_usage = 20000000000;

-- Grant permissions
GRANT SELECT, INSERT ON database.* TO username;

-- Review user permissions
SHOW GRANTS FOR username;
```

## Troubleshooting Guide

### Common Issues

1. **Disk Space Issues**

   - Check disk usage (see [Monitoring Guide](../system-commands/monitoring.md))
   - Remove old data
   - Optimize tables
   - Add new storage

2. **Performance Issues**

   - Review query performance (see [Optimization Guide](optimization.md))
   - Check system resources
   - Optimize table structure
   - Adjust system settings

3. **Replication Issues**

   - Verify network connectivity
   - Check replication queue
   - Restart problematic replicas
   - Re-initialize if necessary

## Recovery Procedures

### Data Recovery

```sql
-- Recover from detached partition
ALTER TABLE database.table ATTACH PARTITION partition_id;

-- Recover from backup
RESTORE TABLE database.table FROM '/path/to/backup';

-- Verify recovery
SELECT count() FROM database.table;
```

### System Recovery

1. Stop ClickHouse service
2. Backup configuration files
3. Check system logs
4. Restore from last known good configuration
5. Start ClickHouse service
6. Verify system health

## Best Practices

1. Regular backups
2. Automated maintenance tasks
3. Proper monitoring setup
4. Documentation of changes
5. Testing in staging environment
6. Regular security updates
7. Capacity planning

## References

- [@ClickHouseSQL Backup and Restore](https://clickhouse.com/docs/en/operations/backup)
- [@Web Administration Guide](https://clickhouse.com/docs/en/operations/administration)
- [@ClickHouseSQL System Tables](https://clickhouse.com/docs/en/operations/system-tables)

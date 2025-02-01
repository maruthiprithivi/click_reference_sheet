# Database Operations

Essential commands for managing databases in ClickHouse Cloud.

## Database Management

### List Databases

```sql
-- Show all databases
SHOW DATABASES;

-- Get detailed database information
SELECT name,
       engine,
       data_path,
       metadata_path,
       uuid
FROM system.databases;
```

### Create Database

```sql
-- Basic database creation
CREATE DATABASE IF NOT EXISTS database_name;

-- Create database with specific engine
CREATE DATABASE IF NOT EXISTS database_name
ENGINE = Atomic;
```

### Drop Database

```sql
DROP DATABASE IF EXISTS database_name;
```

### Switch Database

```sql
USE database_name;
```

## Database Information

### Database Size

```sql
-- Get size of all databases
SELECT
    database,
    formatReadableSize(sum(bytes_on_disk)) AS disk_size,
    count() AS total_tables,
    sum(rows) AS total_rows
FROM system.parts
GROUP BY database
ORDER BY sum(bytes_on_disk) DESC;
```

### Database Metadata

```sql
-- View database engines
SELECT
    name AS database_name,
    engine,
    engine_full,
    uuid
FROM system.databases
ORDER BY name;
```

### Database Permissions

```sql
-- Show database privileges
SHOW GRANTS ON database_name;

-- Check current user's access to databases
SELECT
    database,
    count() AS tables_count,
    any(engine) AS db_engine
FROM system.tables
WHERE has_access_to_database(database)
GROUP BY database;
```

## Monitoring and Maintenance

### Database Activity

```sql
-- Monitor database operations
SELECT
    database,
    event_type,
    event_date,
    event_time,
    query
FROM system.query_log
WHERE type >= 2
  AND event_date >= today() - 1
ORDER BY event_time DESC
LIMIT 10;
```

### Database Performance

```sql
-- Check database metrics
SELECT
    database,
    table,
    formatReadableSize(sum(bytes)) AS memory_usage,
    sum(rows) AS total_rows,
    count() AS part_count
FROM system.parts
GROUP BY database, table
ORDER BY sum(bytes) DESC;
```

## Best Practices

1. Use meaningful database names
2. Implement proper access controls
3. Regular monitoring of database sizes
4. Periodic cleanup of unused databases

## Common Tasks

### Database Backup

```sql
-- Create backup of database structure
SELECT
    concat('CREATE DATABASE IF NOT EXISTS ', name, ' ENGINE = ', engine_full, ';') AS create_statement
FROM system.databases
WHERE name = 'database_name';
```

### Database Replication Status

```sql
-- Check replication status for database tables
SELECT
    database,
    table,
    is_leader,
    total_replicas,
    active_replicas
FROM system.replicas
WHERE database = 'database_name'
ORDER BY table;
```

### Database Settings

```sql
-- View database-specific settings
SELECT
    name,
    value,
    changed,
    description
FROM system.settings
WHERE name LIKE '%database%'
ORDER BY name;
```

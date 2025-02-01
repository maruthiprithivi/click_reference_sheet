# Connection Commands

This section covers essential commands for connecting to and managing ClickHouse Cloud connections.

## Connection Information

### Check Current Connection

```sql
SELECT currentUser() AS current_user,
       currentDatabase() AS current_database,
       currentQueryId() AS query_id;
```

### View Connection Settings

```sql
SELECT name, value
FROM system.settings
WHERE name LIKE '%connection%'
ORDER BY name;
```

## Session Management

### View Active Sessions

```sql
SELECT user,
       client_hostname,
       client_name,
       client_version,
       query_start_time,
       query
FROM system.processes
ORDER BY query_start_time DESC;
```

### Kill a Specific Query

```sql
KILL QUERY WHERE query_id = 'query-id-here';
```

## Connection Pool Information

### Check Connection Pool Status

```sql
SELECT *
FROM system.metrics
WHERE metric LIKE '%Connection%';
```

## Security and Authentication

### View User Privileges

```sql
SHOW GRANTS FOR CURRENT USER;
```

### Check Access Rights

```sql
SELECT * FROM system.grants;
```

## Troubleshooting Tips

### Common Connection Issues

1. **Connection Timeout**

```sql
-- Check network timeouts
SELECT name, value
FROM system.settings
WHERE name LIKE '%timeout%';
```

2. **Connection Limits**

```sql
-- Check max connections
SELECT *
FROM system.metrics
WHERE metric = 'TCPConnection';
```

### Best Practices

1. Always use secure connections (SSL/TLS) in production
2. Set appropriate timeouts for your use case
3. Monitor connection pools for potential issues
4. Use connection pooling when appropriate

## Connection String Examples

```bash
# Basic connection string
clickhouse-client --host=<host> --port=9440 --secure --user=<user> --password=<password>

# Connection with specific database
clickhouse-client --host=<host> --port=9440 --secure --database=<database> --user=<user> --password=<password>
```

## Health Checks

### Basic Health Check Query

```sql
SELECT 1;
```

### Comprehensive Health Check

```sql
SELECT
    currentUser() AS user,
    currentDatabase() AS database,
    version() AS version,
    uptime() AS uptime_seconds,
    timezone() AS timezone;
```

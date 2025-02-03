# Security and Access Control Guide

A comprehensive guide to managing security and access control in ClickHouse.

## Overview

This guide covers essential security features in ClickHouse:

- User management
- Role-based access control (RBAC)
- Row-level security
- SSL/TLS configuration
- Audit logging

[@ClickHouseSQL Reference](https://clickhouse.com/docs/en/operations/access-rights)

## User Management

### Creating and Managing Users

```sql
-- Create a new user with password
CREATE USER john
IDENTIFIED WITH sha256_password BY 'secure_password'
SETTINGS max_memory_usage = 10000000000;  -- 10GB limit

-- Modify user settings
ALTER USER john
SETTINGS max_memory_usage = 20000000000;  -- 20GB limit

-- List all users
SELECT *
FROM system.users
WHERE name NOT LIKE '%default%';
```

### User Quotas

```sql
-- Create a quota
CREATE QUOTA user_quota
    KEYED BY ip_address
    FOR INTERVAL 1 hour MAX queries = 1000
    FOR INTERVAL 1 day MAX result_rows = 1000000;

-- Apply quota to user
ALTER USER john
    QUOTA user_quota;
```

## Role-Based Access Control

### Creating and Managing Roles

```sql
-- Create a role
CREATE ROLE analyst;

-- Grant privileges to role
GRANT SELECT ON database.* TO analyst;
GRANT SELECT ON system.query_log TO analyst;

-- Assign role to user
GRANT analyst TO john;

-- List role grants
SELECT *
FROM system.grants
WHERE user_name = 'john';
```

### Complex Role Management

```sql
-- Create role hierarchy
CREATE ROLE junior_analyst;
CREATE ROLE senior_analyst;

GRANT SELECT ON database.small_tables TO junior_analyst;
GRANT SELECT ON database.* TO senior_analyst;
GRANT junior_analyst TO senior_analyst;

-- Set default role
ALTER USER john DEFAULT ROLE senior_analyst;
```

## Row-Level Security

### Setting Up Row-Level Security

```sql
-- Create row policy
CREATE ROW POLICY department_access
ON database.employees
FOR SELECT
USING department_id = currentUser();

-- Apply policy to role
ALTER ROW POLICY department_access
ON database.employees
TO analyst;
```

### Checking Row Policies

```sql
-- List active policies
SELECT *
FROM system.row_policies
WHERE database = 'database'
  AND table = 'employees';
```

## SSL/TLS Configuration

### Checking SSL Status

```sql
-- Check if connection is secure
SELECT value
FROM system.settings
WHERE name = 'secure';

-- View SSL configuration
SELECT *
FROM system.ssl_certificates;
```

## Audit Logging

### Query Logging

```sql
-- Enable query logging
SET log_queries = 1;

-- View query history
SELECT
    user,
    query_id,
    query,
    exception,
    event_time
FROM system.query_log
WHERE type = 'QueryFinish'
  AND event_date >= today() - 1
ORDER BY event_time DESC;
```

### Access Logging

```sql
-- View access attempts
SELECT
    user_name,
    access_type,
    grant_option,
    database,
    table,
    column,
    creation_time
FROM system.grants
ORDER BY creation_time DESC;
```

## Security Monitoring

### Active Sessions

```sql
-- Monitor current sessions
SELECT
    user,
    client_hostname,
    client_name,
    elapsed,
    query
FROM system.processes
ORDER BY elapsed DESC;
```

### Failed Login Attempts

```sql
-- Check authentication failures
SELECT
    user,
    authentication_method,
    exception,
    event_time
FROM system.query_log
WHERE type = 'ExceptionBeforeStart'
  AND event_date >= today() - 1
  AND exception LIKE '%Authentication%'
ORDER BY event_time DESC;
```

<!-- ## Best Practices

1. Use strong passwords and SHA-256 password hashing
2. Implement principle of least privilege
3. Regularly audit user permissions
4. Enable and monitor query logging
5. Use row-level security for sensitive data
6. Implement proper network security
7. Regular security audits -->

## Troubleshooting

### Common Issues

1. **Permission Denied**

```sql
-- Check user privileges
SELECT *
FROM system.grants
WHERE user_name = 'problem_user';

-- Check role assignments
SELECT *
FROM system.role_grants
WHERE user_name = 'problem_user';
```

2. **Authentication Failures**

```sql
-- Check recent authentication issues
SELECT
    user,
    client_hostname,
    exception
FROM system.query_log
WHERE event_date >= today()
  AND type = 'ExceptionBeforeStart'
  AND exception LIKE '%Authentication%'
ORDER BY event_time DESC;
```

<!--
## References

- [@ClickHouseSQL Security](https://clickhouse.com/docs/en/operations/security)
- [@Web Access Control](https://clickhouse.com/docs/en/operations/access-rights)
- [@ClickHouseSQL System Tables](https://clickhouse.com/docs/en/operations/system-tables/users) -->

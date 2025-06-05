### Table of Contents

- [Introduction](https://www.cloudthat.com/resources/blog/a-guide-to-setup-a-foreign-server-in-amazon-rds-for-postgresql#introduction)
 - [Understanding Foreign Data Wrappers (FDW)](https://www.cloudthat.com/resources/blog/a-guide-to-setup-a-foreign-server-in-amazon-rds-for-postgresql#understanding-foreign-data-wrappers-fdw-)
 - [Prerequisites](https://www.cloudthat.com/resources/blog/a-guide-to-setup-a-foreign-server-in-amazon-rds-for-postgresql#prerequisites)
 - [Step-by-Step Guide](https://www.cloudthat.com/resources/blog/a-guide-to-setup-a-foreign-server-in-amazon-rds-for-postgresql#step-by-step-guide)
 - [Best Practices for Foreign Data Wrappers](https://www.cloudthat.com/resources/blog/a-guide-to-setup-a-foreign-server-in-amazon-rds-for-postgresql#best-practices-for-foreign-data-wrappers)
 - [Advantages of Using Foreign Servers in Amazon RDS PostgreSQL](https://www.cloudthat.com/resources/blog/a-guide-to-setup-a-foreign-server-in-amazon-rds-for-postgresql#advantages-of-using-foreign-servers-in-amazon-rds-postgresql)
 - [Conclusion](https://www.cloudthat.com/resources/blog/a-guide-to-setup-a-foreign-server-in-amazon-rds-for-postgresql#conclusion)

## Introduction

Amazon RDS for PostgreSQL is a managed database service that allows seamless integration with external databases using Foreign Data Wrappers (FDW). FDWs enable PostgreSQL to access data from other databases (PostgreSQL or other sources) as local tables.

### Pioneers in Cloud Consulting & Migration Services

- Reduced infrastructural costs
- Accelerated application deployment

[Get Started](https://www.cloudthat.com/consulting/)

## Understanding Foreign Data Wrappers (FDW)

FDWs in PostgreSQL allow database administrators to query external databases. They provide an abstraction layer, enabling remote data access with minimal performance overhead. Some common use cases include:

- Accessing a remote PostgreSQL database
- Integrating Amazon RDS with on-premise databases
- Querying heterogeneous data sources (e.g., MySQL, Oracle, MongoDB, Redshift)

**PostgreSQL supports multiple FDWs, including:**

- postgres_fdw (for PostgreSQL-to-PostgreSQL connections)
- mysql_fdw (for MySQL integration)
- odbc_fdw (for ODBC-compatible databases)
- For this tutorial, we will focus on postgres_fdw to connect two PostgreSQL databases, both hosted on Amazon RDS.

## Prerequisites

Before configuring a foreign server in Amazon RDS for PostgreSQL, ensure the following:

- Amazon RDS PostgreSQL Instance is up and running.
- Extensions (postgres_fdw) are supported in your PostgreSQL version.
- Security Groups allow connections between the RDS instances.
- A database user with the required privileges is available.

**You can check your PostgreSQL version by running:**

```sql
SELECT version();
```
## Step-by-Step Guide

**Step 1: Enable the postgres_fdw Extension**

First, enable the postgres_fdw extension on the primary database:

```sql
CREATE EXTENSION postgres_fdw;
```

Verify the installation:

```sql
SELECT * FROM pg_extension WHERE extname = 'postgres_fdw';
```

**Step 2: Create a Foreign Server**

A foreign server represents the remote PostgreSQL database. Use the following syntax:

```sql
CREATE SERVER remote_postgres_server  
FOREIGN DATA WRAPPER postgres_fdw  
OPTIONS (host '<REMOTE_DB_ENDPOINT>', port '5432', dbname '<REMOTE_DB_NAME>');
```

**Replace:**

- <REMOTE_DB_ENDPOINT> with the remote RDS PostgreSQL instance endpoint
- <REMOTE_DB_NAME> with the target database name

**To list foreign servers:**

```sql
SELECT * FROM pg_foreign_server;
```

**Step 3: Create a User Mapping**

A user mapping links local users to remote database users. Create it using:

```sql
CREATE USER MAPPING FOR current_user  
SERVER remote_postgres_server  
OPTIONS (user '<REMOTE_USER>', password '<REMOTE_PASSWORD>');
```

Check existing mappings:

```sql
SELECT * FROM pg_user_mappings;
```

For security, avoid storing credentials in SQL scripts and use AWS Secrets Manager instead.

**Step 4: Create a Foreign Table**

A foreign table acts as a proxy to the remote table. Define it as follows:
```sql
CREATE FOREIGN TABLE remote_table (  
    id INT,  
    name TEXT,  
    created_at TIMESTAMP  
)  
SERVER remote_postgres_server  
OPTIONS (schema_name 'public', table_name 'source_table');
```

The columns must match the remote table’s structure.

The schema_name should be the actual schema in the remote database.

To verify:

```sql
SELECT * FROM information_schema.foreign_tables;
```

**Step 5: Querying the Remote Table**

Once the foreign table is set up, query it as a local table:

```sql
SELECT * FROM remote_table LIMIT 10;
```

To ensure performance, analyze query execution:

```sql
EXPLAIN ANALYZE SELECT * FROM remote_table;
```

**Step 6: Managing and Securing Foreign Servers**

Granting Privileges

Grant access to specific users:

```sql
GRANT USAGE ON FOREIGN SERVER remote_postgres_server TO some_user;
GRANT SELECT ON remote_table TO some_user;
```

Updating User Mapping

Modify existing credentials:

```sql
ALTER USER MAPPING FOR current_user  
SERVER remote_postgres_server  
OPTIONS (SET password '<NEW_PASSWORD>');
```

**Dropping Foreign Server or Table**

To remove the foreign table:

```sql
DROP FOREIGN TABLE remote_table;
```

To drop the foreign server:

```sql
DROP SERVER remote_postgres_server CASCADE;
```

## Best Practices for Foreign Data Wrappers

**Optimize Performance:**

- Use LIMIT when querying large datasets.
- Avoid complex joins with foreign tables.
- Consider materialized views for frequently accessed data.

**Secure Connections:**

- Use SSL to encrypt connections between databases.
- Store credentials securely (AWS Secrets Manager, IAM).

**Monitor Query Performance:**

- Use pg_stat_activity to monitor FDW-related queries:
```sql
SELECT * FROM pg_stat_activity WHERE query LIKE '%foreign%';
```

**Use Connection Pooling**

- For frequent queries, consider PgBouncer to manage persistent connections.

## Advantages of Using Foreign Servers in Amazon RDS PostgreSQL

1. **Seamless Data Integration**
    - FDWs allow PostgreSQL to query remote databases as if they were local, reducing the need for complex ETL (Extract, Transform, Load) processes.
2. **Real-Time Access to Remote Data**
    - Instead of duplicating data, you can fetch live records from remote databases, ensuring data consistency across multiple environments.
3. **Reduced Storage Costs**
    - Since FDWs query external databases instead of storing duplicate data, they help save on Amazon RDS storage costs.
4. **Cross-Region and Cross-Account Data Access**
    - Foreign servers enable secure data access between PostgreSQL instances across different AWS accounts or regions.
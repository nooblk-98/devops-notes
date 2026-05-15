# Database Management

## MySQL / MariaDB Installation

```bash
# Ubuntu / Debian
apt install -y mysql-server

# Or MariaDB
apt install -y mariadb-server

# Secure installation
mysql_secure_installation
```

## PostgreSQL Installation

```bash
# Ubuntu / Debian
apt install -y postgresql postgresql-contrib

# Switch to postgres user
sudo -u postgres psql
```

## User Management

### MySQL / MariaDB

```sql
-- Create user
CREATE USER 'appuser'@'localhost' IDENTIFIED BY 'strong_password';

-- Grant privileges
GRANT ALL PRIVILEGES ON appdb.* TO 'appuser'@'localhost';

-- Show users
SELECT User, Host FROM mysql.user;

-- Change password
ALTER USER 'appuser'@'localhost' IDENTIFIED BY 'new_password';

-- Revoke privileges
REVOKE ALL PRIVILEGES ON appdb.* FROM 'appuser'@'localhost';

-- Drop user
DROP USER 'appuser'@'localhost';

FLUSH PRIVILEGES;
```

### PostgreSQL

```sql
-- Create user
CREATE USER appuser WITH PASSWORD 'strong_password';

-- Create database
CREATE DATABASE appdb OWNER appuser;

-- Grant privileges
GRANT ALL PRIVILEGES ON DATABASE appdb TO appuser;

-- List users
\du

-- Change password
ALTER USER appuser WITH PASSWORD 'new_password';

-- Drop user
DROP USER appuser;
```

## Performance Tuning

### MySQL / MariaDB (`/etc/mysql/my.cnf`)

```ini
[mysqld]
innodb_buffer_pool_size = 1G    # 70% of RAM for dedicated DB server
innodb_log_file_size = 256M
innodb_flush_log_at_trx_commit = 2
max_connections = 150
query_cache_type = 0
query_cache_size = 0
tmp_table_size = 64M
max_heap_table_size = 64M
```

### PostgreSQL (`/etc/postgresql/*/main/postgresql.conf`)

```ini
shared_buffers = 256MB           # 25% of RAM
effective_cache_size = 768MB     # 50% of RAM
work_mem = 16MB
maintenance_work_mem = 64MB
random_page_cost = 1.1           # For SSD
```

## Backup & Restore

### MySQL / MariaDB

```bash
# Backup single database
mysqldump -u root -p appdb > appdb-$(date +%Y%m%d).sql

# Backup all databases
mysqldump -u root -p --all-databases > full-backup-$(date +%Y%m%d).sql

# Restore
mysql -u root -p appdb < appdb-20240101.sql
```

### PostgreSQL

```bash
# Backup single database
pg_dump -U postgres appdb > appdb-$(date +%Y%m%d).sql

# Backup all databases
pg_dumpall -U postgres > full-backup-$(date +%Y%m%d).sql

# Restore
psql -U postgres appdb < appdb-20240101.sql
```

## Replication (MySQL)

### Master Config

```ini
[mysqld]
server-id = 1
log_bin = /var/log/mysql/mysql-bin.log
binlog_do_db = appdb
```

### Slave Config

```ini
[mysqld]
server-id = 2
relay-log = /var/log/mysql/mysql-relay-bin.log
```

```bash
# On master
CREATE USER 'replica'@'%' IDENTIFIED BY 'replication_password';
GRANT REPLICATION SLAVE ON *.* TO 'replica'@'%';

# On slave
CHANGE MASTER TO
  MASTER_HOST='master-ip',
  MASTER_USER='replica',
  MASTER_PASSWORD='replication_password',
  MASTER_LOG_FILE='mysql-bin.000001',
  MASTER_LOG_POS=  0;
START SLAVE;
SHOW SLAVE STATUS\G
```

## Common Queries

```sql
-- Show running queries
SHOW FULL PROCESSLIST;

-- Kill a stuck query
KILL QUERY <id>;

-- Show table sizes
SELECT table_schema, table_name, ROUND(data_length/1024/1024, 2) AS 'Size (MB)'
FROM information_schema.tables WHERE table_schema = 'appdb';

-- Slow query log
SHOW VARIABLES LIKE 'slow_query%';
SHOW VARIABLES LIKE 'long_query_time';
```

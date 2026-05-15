# Database Migration

## MySQL Version Upgrade

### Pre-Migration

```bash
# Check current version
mysql --version

# Check storage engines
mysql -e "SELECT TABLE_SCHEMA, TABLE_NAME, ENGINE FROM information_schema.TABLES WHERE ENGINE NOT IN ('InnoDB', 'MyISAM');"

# Check for deprecated features
mysqlcheck -u root -p --all-databases --check-upgrade

# Backup
mysqldump -u root -p --all-databases --routines --triggers --events > pre-upgrade-backup.sql
```

### Upgrade MySQL 5.7 → 8.0

```bash
# Stop MySQL
systemctl stop mysql

# Add MySQL 8.0 repository
apt install -y gnupg
wget -q https://repo.mysql.com/mysql-apt-config_0.8.24-1_all.deb
dpkg -i mysql-apt-config_0.8.24-1_all.deb
apt update

# Upgrade
apt install -y mysql-server

# Run upgrade check
mysql_upgrade -u root -p

# Restart
systemctl restart mysql

# Verify
mysql --version
mysqlcheck -u root -p --all-databases
```

### Character Set & Collation Migration

```sql
-- Check current charset
SHOW VARIABLES LIKE 'character_set_%';
SHOW VARIABLES LIKE 'collation_%';

-- Database level
ALTER DATABASE wordpress CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- Table level
ALTER TABLE wp_posts CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- Column level
ALTER TABLE wp_posts MODIFY post_title TEXT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- All tables in database
SELECT CONCAT('ALTER TABLE ', TABLE_NAME, ' CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;')
FROM information_schema.TABLES
WHERE TABLE_SCHEMA = 'wordpress' AND TABLE_TYPE = 'BASE TABLE';
```

### Migration Script

```bash title="migrate-charset.sh"
#!/bin/bash
DB="wordpress"
USER="root"
PASS="password"

echo "ALTER DATABASE $DB CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;" | mysql -u$USER -p$PASS $DB

TABLES=$(mysql -u$USER -p$PASS $DB -e "SHOW TABLES" | tail -n +2)
for TABLE in $TABLES; do
    echo "Converting $TABLE..."
    echo "ALTER TABLE $TABLE CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;" | mysql -u$USER -p$PASS $DB
done

echo "Done."
```

## PostgreSQL Version Upgrade

```bash
# Check current version
psql --version

# List clusters
pg_lsclusters

# Install new version
apt install -y postgresql-16

# Stop old cluster
pg_ctlcluster 14 main stop

# Upgrade using pg_upgradecluster
pg_upgradecluster 14 main

# Start new cluster
pg_ctlcluster 16 main start

# Verify
psql -V
sudo -u postgres psql -c "SELECT version();"
```

## Migrate Between Database Engines

### MySQL → MariaDB (Direct Upgrade)

```bash
# Stop MySQL
systemctl stop mysql

# Install MariaDB
apt install -y mariadb-server

# Start MariaDB (uses MySQL data directory)
systemctl start mariadb

# Run upgrade
mysql_upgrade -u root -p

# Verify
mysql --version
```

### MariaDB → MySQL

```bash
# Backup from MariaDB
mysqldump -u root -p --all-databases > mariadb-backup.sql

# Install MySQL
apt install -y mysql-server

# Restore
mysql -u root -p < mariadb-backup.sql

# Run mysql_upgrade
mysql_upgrade -u root -p
```

## Cross-Server Migration

```bash
# Direct dump and pipe
mysqldump -u root -p wordpress | mysql -h new-server -u root -p wordpress

# Compressed transfer
mysqldump -u root -p wordpress | gzip | ssh user@new-server "gunzip | mysql -u root -p wordpress"

# Using mysqldump with progress
mysqldump -u root -p --single-transaction --quick wordpress | pv | mysql -h new-server -u root -p wordpress
```

## WordPress Database Migration

```bash
# Export
wp db export wp-backup.sql

# Search & replace URLs
wp search-replace "http://old.com" "https://new.com" --all-tables

# Import on new server
wp db import wp-backup.sql

# Verify
wp db check
```

## Migration Checklist

- [ ] Full backup taken
- [ ] Character set compatibility verified
- [ ] Storage engines compatible
- [ ] Application connection strings updated
- [ ] Post-migration integrity check run
- [ ] Performance tested
- [ ] Rollback plan ready (old version still available)

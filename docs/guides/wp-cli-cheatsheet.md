# WP-CLI Cheat Sheet

## Core

```bash
# Install WordPress
wp core download
wp core config --dbname=wordpress --dbuser=wpuser --dbpass=pass
wp core install --url=example.com --title="My Site" --admin_user=admin --admin_password=pass --admin_email=admin@example.com

# Update
wp core update
wp core update-db

# Verify checksums
wp core verify-checksums
```

## Database

```bash
# Export/import
wp db export backup.sql
wp db import backup.sql

# Optimize tables
wp db optimize

# Repair tables
wp db repair

# Search and replace (safe with serialized data)
wp search-replace "old.com" "new.com" --all-tables
wp search-replace "http://" "https://" --all-tables --skip-columns=guid

# Create database
wp db create
```

## Users

```bash
# Create
wp user create john john@example.com --role=administrator

# List
wp user list

# Update role
wp user update 123 --role=editor

# Delete
wp user delete 123

# Set password
wp user update 123 --user_pass=newpassword

# Generate application password
wp user application-password create 123 myapp
```

## Posts & Content

```bash
# Create post
wp post create --post_title="Hello World" --post_content="Content" --post_status=publish

# List posts
wp post list --posts_per_page=10

# Update post
wp post update 123 --post_title="New Title"

# Delete all posts
wp post delete $(wp post list --format=ids)
```

## Plugins

```bash
# List
wp plugin list

# Install
wp plugin install wordfence --activate

# Activate/deactivate
wp plugin activate wordfence
wp plugin deactivate hello

# Update all
wp plugin update --all

# Delete
wp plugin delete hello

# Check status
wp plugin status
```

## Themes

```bash
# List
wp theme list

# Install
wp theme install twentytwentyfour --activate

# Update
wp theme update --all

# Delete
wp theme delete twentytwentythree

# Switch
wp theme activate twentytwentyfour
```

## Options

```bash
# Get
wp option get siteurl
wp option get home
wp option get blogname

# Update
wp option update blogname "My Site"

# Delete
wp option delete transient_

# List all
wp option list
```

## Cache

```bash
# Flush all caches
wp cache flush

# Redis-specific
wp redis enable
wp redis disable
wp redis status
wp redis info
```

## Maintenance Mode

```bash
# Enable
wp maintenance-mode activate

# Disable
wp maintenance-mode deactivate

# Check status
wp maintenance-mode status
```

## Media

```bash
# Regenerate thumbnails
wp media regenerate

# List images
wp media list --post_mime_type=image/jpeg --posts_per_page=10

# Import from URL
wp media import https://example.com/image.jpg

# Delete unattached media
wp media delete $(wp media list --post_parent=0 --format=ids)
```

## Rewrites

```bash
# Flush permalinks
wp rewrite flush

# List rewrite rules
wp rewrite list

# Structure
wp rewrite structure '/%postname%/'
```

## Cron

```bash
# Run all scheduled cron jobs
wp cron event run --all

# List cron events
wp cron event list

# Add custom cron event
wp cron event schedule wp_version_check hourly

# Run single event
wp cron event run wp_version_check
```

## Multisite

```bash
# List sites
wp site list

# Create site
wp site create --slug=news --title="News"

# Archive
wp site archive 2

# Delete
wp site delete 2

# Switch site
wp site switch 2
```

## Debug

```bash
# Check PHP info
wp --info

# Check environment
wp debug check

# Enable debug mode
wp config set WP_DEBUG true --raw
wp config set WP_DEBUG_LOG true --raw
wp config set WP_DEBUG_DISPLAY false --raw

# View logs
wp debug log

# Tail errors
wp db query "SELECT * FROM wp_options WHERE option_name LIKE '%transient%'" --skip-column-names
```

## Super Admin

```bash
# List super admins
wp super-admin list

# Add
wp super-admin add admin

# Remove
wp super-admin remove admin
```

## Useful One-Liners

```bash
# Disable all plugins
wp plugin deactivate --all

# Enable all plugins
wp plugin activate --all

# Reset password for all users
wp user list --format=ids | xargs -I{} wp user update {} --user_pass=newpass

# Find posts with specific string
wp post list --grep="old-domain" --fields=ID,post_title

# Replace URL in entire network (multisite)
wp search-replace "old.com" "new.com" --network

# Export specific posts by date
wp db export --dbuser=root --dbpass=pass < backup-$(wp eval 'echo date("Y-m-d");').sql

# List all installed languages
wp language core list

# Update all translations
wp language core update
```

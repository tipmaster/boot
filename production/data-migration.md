# Data Migration: /mnt/Data → sin + fue

Migration of database and file data from old server (/mnt/Data on sin) to production (sin) and development (fue).

**Reference:** See [sin-setup.md](sin-setup.md) for server configuration.

---

## Status Summary

| Data | Size | sin | fue | Notes |
|------|------|:---:|:---:|-------|
| **Phase 1** |
| MariaDB | ~10GB | [ ] | [ ] | 10.5.22 → 11.4.9 |
| tmdata | 7.7GB | [ ] | [ ] | Flat files |
| ClickHouse | 528MB | [ ] | [ ] | 2 databases |
| **Phase 2** |
| MongoDB | 1.1GB | [ ] | [ ] | 6.x → 8.0 (version gap) |
| SQLite | ~350MB | [ ] | [ ] | 4 databases |
| Redis | config | [ ] | [ ] | Rebuild from scratch |

---

## Phase 1: MariaDB, tmdata, ClickHouse

### 1. MariaDB (10.5.22 → 11.4.9)

**Source:** `/mnt/Data/var/lib/mysql/`

| Database | Size | Status |
|----------|------|--------|
| gsc | 7.3GB | [ ] |
| posthog | 1.1GB | [ ] |
| forum | 747MB | [ ] |
| strapi4 | 334MB | [ ] |
| mbox | 203MB | [ ] |
| sendy | 136MB | [ ] |
| revive | 51MB | [ ] |
| cloudflare | 49MB | [ ] |
| searchdata | 12MB | [ ] |
| ~~strapi5~~ | 4.3MB | SKIP |

**Migration approach:** mysqldump from old data, restore to running MariaDB 11.4.9

```bash
# On sin - Export from old data directory
# Option A: Start temporary mysqld on old data (complex)
# Option B: Copy data dir, upgrade (requires downtime)

# Recommended: Direct InnoDB file import (same-version compatible)
# Since 10.5 → 11.4 is compatible, we can:

# 1. Stop MariaDB
sudo systemctl stop mariadb

# 2. Copy databases (excluding mysql, performance_schema, strapi5)
sudo cp -a /mnt/Data/var/lib/mysql/gsc /var/lib/mysql/
sudo cp -a /mnt/Data/var/lib/mysql/posthog /var/lib/mysql/
sudo cp -a /mnt/Data/var/lib/mysql/forum /var/lib/mysql/
sudo cp -a /mnt/Data/var/lib/mysql/strapi4 /var/lib/mysql/
sudo cp -a /mnt/Data/var/lib/mysql/mbox /var/lib/mysql/
sudo cp -a /mnt/Data/var/lib/mysql/sendy /var/lib/mysql/
sudo cp -a /mnt/Data/var/lib/mysql/revive /var/lib/mysql/
sudo cp -a /mnt/Data/var/lib/mysql/cloudflare /var/lib/mysql/
sudo cp -a /mnt/Data/var/lib/mysql/searchdata /var/lib/mysql/

# 3. Fix ownership
sudo chown -R mysql:mysql /var/lib/mysql/

# 4. Start MariaDB
sudo systemctl start mariadb

# 5. Run upgrade check
sudo mariadb-upgrade

# 6. Verify
mysql -e "SHOW DATABASES;"
```

**Verification:**
```bash
mysql -e "SELECT table_schema, COUNT(*) as tables FROM information_schema.tables WHERE table_schema NOT IN ('mysql','information_schema','performance_schema') GROUP BY table_schema;"
```

---

### 2. tmdata (Flat Files)

**Source:** `/mnt/Data/opt/tmdata/` (7.7GB)
**Target:** `/opt/tmdata/`

```bash
# On sin - rsync preserving permissions
sudo rsync -avP /mnt/Data/opt/tmdata/ /opt/tmdata/

# Fix ownership
sudo chown -R deploy:nginx /opt/tmdata/

# Verify
ls -la /opt/tmdata/
du -sh /opt/tmdata/
```

**Key files to verify:**
- hashedPasswords.txt
- blanko.txt
- tmi/ directory
- btm/ directory

---

### 3. ClickHouse

**Source:** `/mnt/Data/var/lib/clickhouse/`

| Database | Tables | Status |
|----------|--------|--------|
| crawlanalyzer_indexing | indexing_submissions, quota_usage, sites, url_inspection_results, url_lists | [ ] |
| gsc_center | collection_log, gsc_data, sites | [ ] |
| ~~default~~ | (empty) | SKIP |
| ~~system~~ | (built-in) | SKIP |

**Migration approach:** Copy data + metadata directories

```bash
# On sin
# 1. Stop ClickHouse
sudo systemctl stop clickhouse-server

# 2. Copy databases (data + metadata + store)
sudo cp -a /mnt/Data/var/lib/clickhouse/data/crawlanalyzer_indexing /var/lib/clickhouse/data/
sudo cp -a /mnt/Data/var/lib/clickhouse/data/gsc_center /var/lib/clickhouse/data/
sudo cp -a /mnt/Data/var/lib/clickhouse/metadata/crawlanalyzer_indexing /var/lib/clickhouse/metadata/
sudo cp -a /mnt/Data/var/lib/clickhouse/metadata/crawlanalyzer_indexing.sql /var/lib/clickhouse/metadata/
sudo cp -a /mnt/Data/var/lib/clickhouse/metadata/gsc_center /var/lib/clickhouse/metadata/
sudo cp -a /mnt/Data/var/lib/clickhouse/metadata/gsc_center.sql /var/lib/clickhouse/metadata/

# Copy relevant store UUIDs (check metadata for UUIDs)
# May need to copy specific store/<uuid> directories

# 3. Fix ownership
sudo chown -R clickhouse:clickhouse /var/lib/clickhouse/

# 4. Start ClickHouse
sudo systemctl start clickhouse-server

# 5. Verify
clickhouse-client -q "SHOW DATABASES;"
clickhouse-client -q "SELECT database, name FROM system.tables WHERE database IN ('crawlanalyzer_indexing', 'gsc_center');"
```

---

## Phase 2: MongoDB, SQLite, Redis

### MongoDB (6.x → 8.0)

**Database:** `sportmonks`
**Collections (16):** aigcore, autocomplete_responses, countries, fixtures, keywords, leagues, organic_responses, players, queries, seasons, standings, teams, tokens, topscorers, trends_responses, tvstations

**Warning:** Version gap (6.x → 8.0) requires mongodump/mongorestore, not direct file copy.

```bash
# Approach: Start temporary mongod on old data, dump, restore

# 1. Start temporary mongod on port 27018 with old data
mongod --dbpath /mnt/Data/var/lib/mongo --port 27018 --bind_ip 127.0.0.1 &

# 2. Dump sportmonks database
mongodump --port 27018 --db sportmonks --out /tmp/mongodump/

# 3. Stop temporary mongod
kill %1

# 4. Restore to running MongoDB 8.0
mongorestore --db sportmonks /tmp/mongodump/sportmonks/

# 5. Verify
mongosh --eval "db.getSiblingDB('sportmonks').getCollectionNames()"
```

---

### SQLite

| Source | Target |
|--------|--------|
| /mnt/Data/opt/strapi3/.tmp/data.db | /opt/strapi3/.tmp/data.db |
| /mnt/Data/opt/athlete-training/backend/data/training.db | /opt/athlete-training/backend/data/training.db |
| /mnt/Data/opt/lt/crawlanalyzer/data/crawlanalyzer.db | /opt/lt/crawlanalyzer/data/crawlanalyzer.db |
| /mnt/Data/opt/knowledge-monitor/state/knowledge.db | /opt/knowledge-monitor/state/knowledge.db |

```bash
# Copy SQLite files (simple file copy)
cp /mnt/Data/opt/strapi3/.tmp/data.db /opt/strapi3/.tmp/
cp /mnt/Data/opt/athlete-training/backend/data/training.db /opt/athlete-training/backend/data/
cp /mnt/Data/opt/lt/crawlanalyzer/data/crawlanalyzer.db /opt/lt/crawlanalyzer/data/
cp /mnt/Data/opt/knowledge-monitor/state/knowledge.db /opt/knowledge-monitor/state/

# Fix ownership
chown deploy:nginx /opt/strapi3/.tmp/data.db
chown deploy:clickhouse /opt/athlete-training/backend/data/training.db
chown deploy:clickhouse /opt/lt/crawlanalyzer/data/crawlanalyzer.db
chown deploy:clickhouse /opt/knowledge-monitor/state/knowledge.db
```

---

### Redis (Config Only)

**No data migration** - rebuild from scratch.

Key config settings to apply:
```
bind 0.0.0.0
port 6379
databases 16
save 900 1
appendfsync everysec
```

---

## Migration to fue (Dev Server)

After sin migrations are verified, replicate to fue:

```bash
# From sin to fue via rsync over SSH
rsync -avP --progress /opt/tmdata/ deploy@fedora.flywheel.bz:/opt/tmdata/

# For databases: dump on sin, transfer, restore on fue
# (fue needs database servers installed first)
```

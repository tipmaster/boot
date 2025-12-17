# Data Migration: /mnt/Data → sin + fue

Migration of database and file data from old server (/mnt/Data on sin) to production (sin) and development (fue).

**Reference:** See [sin-setup.md](sin-setup.md) for server configuration.

---

## Status Summary

| Data | Size | sin | fue | Notes |
|------|------|:---:|:---:|-------|
| **Phase 1** |
| MariaDB | ~10GB | [x] | [ ] | 9 databases migrated |
| tmdata | 8.5GB | [x] | [ ] | Flat files |
| ClickHouse | 528MB | [x] | [ ] | 2 databases (16M+ rows) |
| **Phase 2** |
| MongoDB | 1.1GB | [x] | [ ] | 4 databases migrated (99%+) |
| SQLite | ~350MB | [x] | [x] | 3 databases (strapi3, crawlanalyzer, knowledge-monitor) |
| Redis | config | [x] | [x] | Configured from scratch |

---

## Phase 1: MariaDB, tmdata, ClickHouse

### 1. MariaDB (10.5.22 → 11.4.9)

**Source:** `/mnt/Data/var/lib/mysql/`

| Database | Size | Status |
|----------|------|--------|
| gsc | 7.3GB | [x] Migrated |
| posthog | 1.1GB | [x] Migrated |
| forum | 747MB | [x] Migrated |
| strapi4 | 334MB | [x] Migrated |
| mbox | 203MB | [x] Migrated |
| sendy | 136MB | [x] Migrated |
| revive | 51MB | [x] Migrated |
| cloudflare | 49MB | [x] Migrated |
| searchdata | 12MB | [x] Migrated |
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

### 2. tmdata (Flat Files) ✅ COMPLETED on sin

**Source:** `/mnt/Data/opt/tmdata/` (7.7GB → 8.5GB)
**Target:** `/opt/tmdata/`
**Migration completed:** 2025-12-17

```bash
# On sin - rsync preserving permissions
sudo rsync -avP /mnt/Data/opt/tmdata/ /opt/tmdata/

# Fix ownership
sudo chown -R deploy:nginx /opt/tmdata/

# Verify
ls -la /opt/tmdata/
du -sh /opt/tmdata/  # 8.5G
```

**Verified directories:**
- tmi/ ✓
- btm/ ✓
- hashedPasswords.txt ✓
- blanko.txt ✓

---

### 3. ClickHouse ✅ COMPLETED on sin

**Source:** `/mnt/Data/var/lib/clickhouse/`
**Migration completed:** 2025-12-17

| Database | Table | Rows |
|----------|-------|------|
| crawlanalyzer_indexing | indexing_submissions | 517 |
| crawlanalyzer_indexing | quota_usage | 2 |
| crawlanalyzer_indexing | sites | 239 |
| crawlanalyzer_indexing | url_inspection_results | 68,383 |
| crawlanalyzer_indexing | url_lists | 50,603 |
| crawlanalyzer_indexing | (5 views) | - |
| gsc_center | collection_log | 201,211 |
| gsc_center | gsc_data | 16,186,984 |
| gsc_center | sites | 239 |

**Store UUIDs migrated:**
- crawlanalyzer_indexing: `90f4d398-7de2-43cd-a587-57a6acec7951`
- gsc_center: `a24d56eb-cad6-4cf0-9eca-b17c2a4d6946`

```bash
# Migration commands used:
sudo systemctl stop clickhouse-server
sudo cp -a /mnt/Data/var/lib/clickhouse/data/{crawlanalyzer_indexing,gsc_center} /var/lib/clickhouse/data/
sudo cp -a /mnt/Data/var/lib/clickhouse/metadata/crawlanalyzer_indexing* /var/lib/clickhouse/metadata/
sudo cp -a /mnt/Data/var/lib/clickhouse/metadata/gsc_center* /var/lib/clickhouse/metadata/
sudo mkdir -p /var/lib/clickhouse/store/{90f,a24}
sudo cp -a /mnt/Data/var/lib/clickhouse/store/90f/90f4d398-7de2-43cd-a587-57a6acec7951 /var/lib/clickhouse/store/90f/
sudo cp -a /mnt/Data/var/lib/clickhouse/store/a24/a24d56eb-cad6-4cf0-9eca-b17c2a4d6946 /var/lib/clickhouse/store/a24/
sudo chown -R clickhouse:clickhouse /var/lib/clickhouse/
sudo systemctl start clickhouse-server
```

---

## Phase 2: MongoDB, SQLite, Redis

### MongoDB ✅ COMPLETED on sin (6.x → 8.0)

**Migration completed:** 2025-12-17

**Found 4 databases** (not just sportmonks):

| Database | Collections | Docs Migrated |
|----------|-------------|---------------|
| sportmonks | 10 | ~170K (99%+) |
| seofordata | 6 | 214K (100%) |
| aig | 1 | 4.5K (100%) |
| smv3 | 13 | 102K (100%) |

**sportmonks detail:**
| Collection | Migrated | Note |
|------------|----------|------|
| teams | 46,322 | 99.99% |
| fixtures | 76,000 | 89% (BSON corruption) |
| seasons | 17,384 | 100% |
| players | 6,853 | 100% |
| standings | 5,400 | 100% |
| queries | 5,448 | 100% |
| leagues | 2,381 | 100% |
| tvstations | 984 | 100% |
| countries | 243 | 100% |

**Migration approach:**
```bash
# 1. Fix ownership and start temp mongod on old data
sudo chown -R mongod:mongod /mnt/Data/var/lib/mongo/
sudo mongod --dbpath /mnt/Data/var/lib/mongo --port 27018 --bind_ip 127.0.0.1 --fork --logpath /tmp/mongod-old.log

# 2. Dump all databases
mongodump --port 27018 --db sportmonks --out /tmp/mongodump/
mongodump --port 27018 --db seofordata --out /tmp/mongodump/
mongodump --port 27018 --db aig --out /tmp/mongodump/
mongodump --port 27018 --db smv3 --out /tmp/mongodump/

# 3. Restore to MongoDB 8.0
mongorestore --db sportmonks /tmp/mongodump/sportmonks/
mongorestore --db seofordata /tmp/mongodump/seofordata/
mongorestore --db aig /tmp/mongodump/aig/
mongorestore --db smv3 /tmp/mongodump/smv3/

# 4. Cleanup
sudo pkill -f 'mongod.*27018'
rm -rf /tmp/mongodump
```

---

### SQLite ✅ COMPLETED

| Source | Size | sin | fue |
|--------|------|:---:|:---:|
| strapi3/.tmp/data.db | 344MB | [x] | [x] |
| lt/crawlanalyzer/data/crawlanalyzer.db | 500KB | [x] | [x] |
| knowledge-monitor/state/knowledge.db | 5.5MB | [x] | [x] |
| ~~athlete-training/backend/data/training.db~~ | - | SKIP | SKIP |

**Note:** athlete-training SQLite was not present in source data.

**Migration completed:** 2025-12-17

```bash
# On sin - copy from /mnt/Data/opt
cp /mnt/Data/opt/strapi3/.tmp/data.db /opt/strapi3/.tmp/
cp /mnt/Data/opt/lt/crawlanalyzer/data/crawlanalyzer.db /opt/lt/crawlanalyzer/data/
cp /mnt/Data/opt/knowledge-monitor/state/knowledge.db /opt/knowledge-monitor/state/

# On fue - rsync from sin
rsync -avz -e "ssh -p 42109" sin:/opt/strapi3/.tmp/data.db /opt/strapi3/.tmp/
rsync -avz -e "ssh -p 42109" sin:/opt/lt/crawlanalyzer/data/crawlanalyzer.db /opt/lt/crawlanalyzer/data/
rsync -avz -e "ssh -p 42109" sin:/opt/knowledge-monitor/state/knowledge.db /opt/knowledge-monitor/state/
```

---

### Redis ✅ COMPLETED

**No data migration** - configured from scratch on both servers.

| Server | Version | Status |
|--------|---------|--------|
| sin | 8.4.0 | [x] Running |
| fue | 8.4.0 | [x] Running (compiled from source) |

Key config settings applied:
```
bind 127.0.0.1
port 6379
databases 16
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

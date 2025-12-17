# Changelog

## 2025-12-17

### Cron.d Ownership Fix (sin + fue)
- **Problem**: `/etc/cron.d` files must be owned by `root:root` - symlinks don't work
- **Solution**: Copy files instead of symlinking with a sync script
- Created `/opt/infra-config/scripts/sync-cron-files.sh`
- Removed broken symlinks from `/opt/serverconfig/etc/cron.d/`
- Run after adding/changing project cron files: `/opt/infra-config/scripts/sync-cron-files.sh`
- Note: nginx and systemd configs work fine with deploy-owned symlinks (only need read access)

### MRTG Server Monitoring (sin)
- Migrated MRTG from old server to `/opt/mrtg` (tipmaster/mrtg repo)
- 45 monitoring metrics across 11 categories:
  - CPU, Memory, Disk, Network, MariaDB, MongoDB, Redis, Nginx, PHP-FPM, Perl, System
- Fixed scripts for AlmaLinux 9 compatibility:
  - MariaDB: unix_socket auth via `mariadb` command (no passwords)
  - Network: /proc/net/dev instead of iftop (no root required)
  - Interface: eth0 → enp6s0 for modern naming
- Installed packages: sysstat (iostat), iftop
- Deploy structure: deploy/nginx/mrtg.flywheel.bz.conf, deploy/mrtg.cron
- Cron job: every 3 minutes via /etc/cron.d/mrtg
- Output: /opt/mrtg/output/ (187 PNG, 47 HTML files)
- Domain: https://mrtg.flywheel.bz/ (Cloudflare Flexible SSL → origin HTTP:80)

### Gitignored .env File Sync (sin + local)
- Created sync script `/opt/infra-config/scripts/sync-env-files.sh`
- Synced `.env`, `.env.shared`, and `credentials.json` files from `/mnt/Data/opt/` to `/opt/`
- Files synced on sin server and locally for 14 projects
- Documented shared credentials in `infra-config/.env.shared` (AI APIs, databases, Cloudflare, etc.)
- Projects: athlete-training, crawlanalyzer, dataforseo, gsc-center, inbox-cleanup, knowledge-monitor, lt, serp2rank, sportmonks-embed, strapi3, strapi4, strapi5, tmapp

### Sendy & Revive Fix (sin)
- Fixed nginx config for sendy.flywheel.bz and ads.flywheel.bz
- Both configs were pointing to port 9000 (ClickHouse) instead of PHP-FPM
- Sendy: port 9002 (PHP 8.1 FPM) - 200 OK
- Revive: port 9002 (PHP 8.1 FPM) - 200 OK
- tmcommunity/vBulletin: port 9003 (PHP 5.6 FPM) - required
- Copied var/ files from /mnt/Data for Revive config

### Git Branch Cleanup
- Deleted stale `master` branches from: tmcommunity, crawlanalyzer
- Deleted stale `main1` branch from serp2rank
- Fixed `origin/HEAD` to point to `main` in 13 repos
- All tipmaster repos now use `main` as default branch

### MariaDB Unix Socket Authentication (sin + fue)
- Configured passwordless auth for `deploy` user via unix_socket plugin
- No password needed - authenticates via Linux user identity (secure, local-only)
- Full admin privileges for development/maintenance

### Data Migration: SQLite Files (sin + fue)
- Copied 3 SQLite databases from /mnt/Data to both sin and fue
- strapi3/.tmp/data.db (344MB)
- lt/crawlanalyzer/data/crawlanalyzer.db (500KB)
- knowledge-monitor/state/knowledge.db (5.5MB)
- athlete-training SQLite not present in source (skipped)
- tmdata migration verified complete on sin (8.5GB)

### Data Migration: ClickHouse (sin)
- Migrated 2 databases: crawlanalyzer_indexing, gsc_center
- Total rows: 16.5M+ (gsc_data: 16.2M, collection_log: 201K)
- Copied data/, metadata/, and store/ directories with UUIDs

### Data Migration: MongoDB (sin)
- Migrated 4 databases: sportmonks, seofordata, aig, smv3
- Used temp mongod on port 27018 → mongodump → mongorestore
- Total: ~490K documents (99%+ success)
- Minor BSON corruption in sportmonks.fixtures (89% recovered)

### Fedora Dev Environment (fue) Package Parity
- Installed vim 9.1 and htop 3.4.1 to match production baseline
- Replaced Valkey 8.1.5 with Redis 8.4.0 (compiled from source) for version parity with production
- Created systemd service for Redis (`/etc/systemd/system/redis.service`)
- Configured ClickHouse with proper config (users_config, default_profile)
- Updated bootstrap dependencies script:
  - Removed lighttpd (using nginx instead)
  - Removed certbot (SSL handled differently)
  - Added redis to package list
- Installed PHP 8.1.33 via Remi modular repo
- Installed Postfix 3.10.3
- Installed Yarn 1.22.22 globally
- Upgraded nginx 1.28.0 → 1.29.4 (nginx mainline repo)
- Installed nvm 0.40.2 with Node.js 14.21.3, 18.20.8, 22.21.1
- Installed Playwright 1.57.0
- Documented cross-system version gaps (Fedora 43 vs AlmaLinux 9)
  - MariaDB 11.4 unavailable on Fedora (boost ABI incompatibility)
  - System tools (git, vim, gcc, etc.) newer on Fedora by design

### Production Server (sin) Setup
- Completed initial server setup (13 steps)
- Hardware: Intel i9-9900K, 64GB RAM, 3x 238GB SSDs (RAID1)
- OS: AlmaLinux 9.7
- Created deploy user with passwordless sudo
- Configured SSH (port 42109, key-based auth)
- Configured firewall (removed port 22, kept 42109)
- Set timezone to Europe/Berlin
- Enabled EPEL, CRB repos and Node.js 22 module
- Installed base packages: git 2.47.3, vim 8.2, htop 3.3.0, make 4.3, gcc 11.5.0, node 22.19.0, npm 10.9.3
- Disabled SELinux
- Documented full hardware specs and RAID configuration

## 2025-12-16

### Initial Setup
- Added Fedora development environment setup guide
- Added context-engineering symlinks
- Initial commit with fedora directory structure

# Production Server Setup: sin

## Server Hardware Specifications

### CPU
| Property | Value |
|----------|-------|
| Model | Intel Core i9-9900K @ 3.60GHz |
| Architecture | x86_64 |
| Sockets | 1 |
| Cores | 8 |
| Threads | 16 (2 per core) |

### Memory
| Type | Size |
|------|------|
| RAM | 64 GB |
| Swap | 16 GB |

### Storage

#### Physical Disks
| Device | Size | Purpose |
|--------|------|---------|
| sda | 238.5 GB | System disk 1 (RAID) |
| sdb | 238.5 GB | System disk 2 (RAID) |
| sdc | 238.5 GB | Data disk (old server image) |

#### RAID Configuration (Software RAID1 - Mirrored)
| Device | Size | Filesystem | Mount | Status |
|--------|------|------------|-------|--------|
| md124 | 1 GB | vfat | /boot/efi | [UU] OK |
| md125 | 2 GB | xfs | /boot | [UU] OK |
| md126 | 220 GB | xfs | / | [UU] OK |
| md127 | 16 GB | swap | [SWAP] | [UU] OK |

#### Data Drive (Non-RAID)
| Device | Size | Filesystem | Mount | Used |
|--------|------|------------|-------|------|
| sdc2 | 220 GB | ext4 | /mnt/Data | 149 GB (73%) |

### Network
| Interface | IP | Purpose |
|-----------|-----|---------|
| enp6s0 | 101.99.89.98/24 | Primary |
| lo | 127.0.0.1/8 | Loopback |

## Server Details

| Property | Value |
|----------|-------|
| Hostname | sin |
| IP | 101.99.89.98 |
| SSH Port | 42109 |
| OS | AlmaLinux 9.7 (Moss Jungle Cat) |
| Kernel | 5.14.0-611.13.1.el9_7.x86_64 |

## Connection

```bash
ssh -p 42109 deploy@sin
ssh -p 42109 root@sin
```

## Credentials

| User | Password | Sudo |
|------|----------|------|
| deploy | sin.prod.2025 | NOPASSWD |
| root | (key-based only) | - |

## Installed Software

### Runtimes
| Runtime | Version | Notes |
|---------|---------|-------|
| Perl | 5.42.0 | Compiled from source with threads |
| Python | 3.13.1 | Compiled from source |
| Node.js | 22.21.1 | Primary (via nvm) |
| Node.js | 18.20.8 | Strapi 4 (via nvm) |
| Node.js | 14.21.3 | Strapi 3 EOL (via nvm) |
| PHP | 8.1.34 | Remi modular (Sendy, Revive) |
| PHP | 5.6.40 | Compiled from source for vBulletin (/usr/local/php56) |

### Databases
| Database | Version | Service |
|----------|---------|---------|
| MariaDB | 11.4.9 LTS | mariadb |
| MongoDB | 8.0.16 | mongod |
| ClickHouse | 25.11.2 | clickhouse-server |
| Redis | 8.4.0 | redis |
| SQLite | 3.34.1 | - |

### Web Stack
| Software | Version | Service | Port | Used by |
|----------|---------|---------|------|---------|
| nginx | 1.29.4 | nginx | 80/443 | - |
| PHP-FPM | 8.1.34 | php-fpm | 9002 | Sendy, Revive |
| PHP-FPM | 5.6.40 | php56-fpm | 9003 | tmcommunity/vBulletin |
| Postfix | 3.5.25 | postfix | 25 | - |

### Tools
| Tool | Version |
|------|---------|
| nvm | 0.40.2 |
| npm | 10.9.4 |
| Yarn | 1.22.22 |
| PM2 | 6.0.14 |
| Playwright | 1.57.0 |
| cpanm | 1.7048 |

### Enabled Repositories
- EPEL 9
- CRB (CodeReady Builder)
- Remi (modular + safe)
- MariaDB 11.4
- MongoDB 8.0
- ClickHouse stable
- nginx mainline

---

## Setup Steps Completed

### 1. Created deploy user with wheel group
```bash
useradd -m -s /bin/bash deploy
usermod -aG wheel deploy
```

### 2. Set up SSH authorized_keys for deploy user
- 14 keys from serverconfig project
- Location: `/home/deploy/.ssh/authorized_keys`

### 3. Set up SSH authorized_keys for root user
- 14 keys from serverconfig project
- Location: `/root/.ssh/authorized_keys`

### 4. Set deploy user password
```bash
passwd deploy
```
- Password: `sin.prod.2025`

### 5. Changed hostname to sin (persistent)
```bash
hostnamectl set-hostname sin
```

### 6. Added sin to local hosts file (on dev machine)
```bash
sudo sh -c 'echo "101.99.89.98 sin" >> /etc/hosts'
```

### 7. Configured firewall
- Removed default ssh service (port 22)
- Kept cockpit service (port 9090) for later decision
- Custom SSH port 42109/tcp allowed
```bash
firewall-cmd --remove-service=ssh --permanent
firewall-cmd --reload
```

### 8. Set up passwordless sudo for deploy user
```bash
echo "deploy ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/deploy
chmod 440 /etc/sudoers.d/deploy
```

### 9. Set timezone to Europe/Berlin
```bash
timedatectl set-timezone Europe/Berlin
```

### 10. Enabled EPEL and CRB repositories
```bash
dnf install -y epel-release
/usr/bin/crb enable
```

### 11. Enabled Node.js 22 module stream
```bash
dnf module enable nodejs:22 -y
```

### 12. Installed base packages
```bash
dnf install -y git vim-enhanced htop make gcc nodejs npm
```

### 13. Disabled SELinux and rebooted
```bash
sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config
reboot
```
- Verified: `getenforce` returns `Disabled`

### 14. Installed production software stack
- Compiled Perl 5.42.0 from source (`-Dusethreads -Duse64bitall`)
- Compiled Python 3.13.1 from source with pip
- Installed nvm 0.40.2 with Node.js 14/18/22
- Installed databases: MariaDB 11.4.9, MongoDB 8.0.16, ClickHouse 25.11.2, Redis 8.4.0
- Installed web stack: nginx 1.29.4, PHP 8.1.34 (FPM), Postfix 3.5.25
- Installed tools: Yarn, PM2 6.0.14, Playwright 1.57.0, cpanm
- All database services enabled and running

### 14a. Configured MariaDB unix_socket authentication
```bash
# Enable unix_socket auth for root and deploy user (run in safe mode or as root)
mariadb -e "
  FLUSH PRIVILEGES;
  ALTER USER 'root'@'localhost' IDENTIFIED VIA unix_socket;
  CREATE USER IF NOT EXISTS 'deploy'@'localhost' IDENTIFIED VIA unix_socket;
  GRANT ALL PRIVILEGES ON *.* TO 'deploy'@'localhost' WITH GRANT OPTION;
  FLUSH PRIVILEGES;
"
```
- **No password required** - authenticates via Linux user identity
- Only works locally (secure by design)
- deploy user has full admin access for development/maintenance

### 15. Compiled PHP 7.1.33 from source for vBulletin
PHP 7.1 EOL Dec 2019, not available in Remi for EL9. Built without OpenSSL due to OpenSSL 3.x incompatibility.

**Build configuration:**
```bash
./configure \
  --prefix=/usr/local/php71 \
  --with-config-file-path=/usr/local/php71/etc \
  --enable-fpm --with-fpm-user=nginx --with-fpm-group=nginx \
  --enable-mysqlnd --with-mysqli=mysqlnd --with-pdo-mysql=mysqlnd \
  --with-curl --with-gd --with-jpeg-dir=/usr --with-png-dir=/usr --with-freetype-dir=/usr \
  --with-zlib --enable-mbstring --enable-xml --enable-simplexml --enable-session \
  --enable-json --enable-opcache --enable-zip --with-libzip \
  --enable-sockets --enable-bcmath --disable-phar
```

**Installation paths:**
- Binary: `/usr/local/php71/bin/php`
- FPM: `/usr/local/php71/sbin/php-fpm`
- Config: `/usr/local/php71/etc/php.ini`
- FPM config: `/usr/local/php71/etc/php-fpm.conf`
- Pool config: `/usr/local/php71/etc/php-fpm.d/www.conf`

**Enabled extensions:**
bcmath, ctype, curl, date, dom, fileinfo, filter, gd, hash, iconv, json, libxml, mbstring, mysqli, mysqlnd, pcre, PDO, pdo_mysql, pdo_sqlite, posix, Reflection, session, SimpleXML, sockets, SPL, sqlite3, standard, tokenizer, xml, xmlreader, xmlwriter, zip, zlib

**FPM Pool settings** (matched from old server):
- user: deploy, group: nginx
- listen: 127.0.0.1:9001
- pm.max_children: 50, start_servers: 5, min_spare: 5, max_spare: 35
- slowlog: /var/log/php71-fpm-slow.log
- session.save_path: /var/lib/php/session

**Service:**
```bash
systemctl status php71-fpm  # Check status
systemctl restart php71-fpm # Restart
```

### 16. Synced gitignored .env and credential files from /mnt/Data/opt

Project `.gitignore` files exclude sensitive files (`.env`, `.env.shared`, `credentials.json`) that must be copied manually from the old server data.

**Source of truth:** `/mnt/Data/opt/` (old server filesystem)

**Sync script:** `/opt/infra-config/scripts/sync-env-files.sh`

```bash
# On sin server: sync from /mnt/Data/opt to /opt
/opt/infra-config/scripts/sync-env-files.sh sin-internal

# From local machine: sync from sin:/mnt/Data/opt to local /opt
/opt/infra-config/scripts/sync-env-files.sh sin-to-local

# List all files and their sync status
/opt/infra-config/scripts/sync-env-files.sh list
```

**Files synced:**

| Project | Files |
|---------|-------|
| `infra-config` | `.env.shared` (shared API keys for all projects) |
| `athlete-training` | `backend/.env`, `backend/.env.mcp`, `backend/.env.test` |
| `crawlanalyzer` | `.env` |
| `dataforseo` | `.env` |
| `gsc-center` | `.env` |
| `inbox-cleanup` | `.env`, `credentials.json` |
| `knowledge-monitor` | `.env` |
| `lt` | `.env` |
| `serp2rank` | `.env` |
| `sportmonks-embed` | `.env` |
| `strapi3` | `config/.env` |
| `strapi4` | `.env`, `.env.dev`, `.env.prod`, `.env.stg`, `.env.tst` |
| `strapi5` | `.env` |
| `tmapp` | `.env` |

**Shared credentials in `.env.shared`:**
- AI APIs: OpenAI, Anthropic, Gemini, xAI, OpenRouter, Perplexity
- SEO: DataForSEO
- Cloudflare: API token, Account ID
- Databases: MongoDB, ClickHouse, Redis connection params
- Sports/Fitness: Sportmonks, Garmin, Intervals.icu
- Other: DeepL, NameSilo

**Usage in projects:**
```bash
# Source shared credentials in project .env or scripts
source /opt/infra-config/.env.shared
# Or use set -a for export
set -a && source /opt/infra-config/.env.shared && set +a
```

### 17. Configured PM2 apps and auto-restart on server reboot

**PM2 Apps Running:**

| App | Project | Node Version | Port | Notes |
|-----|---------|--------------|------|-------|
| strapi3 | /opt/strapi3 | 14.21.3 | 1337 | Uses wrapper script for nvm |
| strapi4 | /opt/strapi4 | 22.21.1 | 1338 | - |
| promos | /opt/promos | 22.21.1 | 3000 | Cluster mode |
| promos-admin | /opt/promos | 22.21.1 | 3001 | Cluster mode |

**Strapi 3 Special Configuration:**

Strapi 3 requires Node.js ≤14 and has dependency issues with newer Node versions:

```bash
# Wrapper script to use Node 14 via nvm
cat /opt/strapi3/start.sh
#!/bin/bash
export NVM_DIR="$HOME/.nvm"
source "$NVM_DIR/nvm.sh"
nvm use 14
cd /opt/strapi3
exec node node_modules/.bin/strapi develop

# Fix duplicate graphql module issue
rm -rf /opt/strapi3/node_modules/strapi-plugin-graphql/node_modules/graphql

# Required peer dependencies (install with Node 14)
nvm use 14
npm install graphql@15 knex@0.21.18 sqlite3
```

**Strapi 4 Configuration:**

```bash
# Copy database config template
cp /opt/strapi4/config/database.js.dev /opt/strapi4/config/database.js
```

**PM2 Startup Configuration:**

```bash
# Save current process list
pm2 save

# Generate and install systemd startup script
pm2 startup
# Then run the sudo command it outputs:
sudo env PATH=$PATH:/home/deploy/.nvm/versions/node/v22.21.1/bin \
  /home/deploy/.nvm/versions/node/v22.21.1/lib/node_modules/pm2/bin/pm2 \
  startup systemd -u deploy --hp /home/deploy

# Verify
systemctl is-enabled pm2-deploy  # Should show: enabled
cat /home/deploy/.pm2/dump.pm2 | python3 -c "import json,sys; d=json.load(sys.stdin); print([p['name'] for p in d])"
# Output: ['strapi3', 'strapi4', 'promos', 'promos-admin']
```

**PM2 Commands:**

```bash
pm2 status              # View all apps
pm2 logs <app>          # View logs
pm2 restart <app>       # Restart specific app
pm2 save                # Save current state (run after changes)
```

---

## Pending Tasks

### Completed (2025-12-17)
- [x] Clone repositories to /opt/
- [x] Set up symlinks per bootstrap pattern
  - Root-level symlinks: 13 created
  - Serverconfig /etc symlinks: 12 created
  - Context-engineering symlinks: via set-soft-links.sh
  - Legacy paths removed: code updated to use `/lt/`
  - ClickHouse: updated to use `/usr/bin/clickhouse` and default ports

### Next
- [ ] Migrate remaining nginx site configs
- [ ] Create Cloudflare DNS A record for mrtg.flywheel.bz → 101.99.89.98 (token needs DNS edit permissions)

### Completed (2025-12-17 continued)
- [x] Configure PHP 8.1 FPM pool
  - Symlinked `/etc/php-fpm.d/www.conf` → `/opt/serverconfig/etc/php-fpm.d/www.conf`
  - User: deploy, Port: 9002 (9000 used by ClickHouse, 9001 by PHP 7.1)
- [x] Configure PM2 apps and auto-restart
  - Started: strapi3, strapi4, promos, promos-admin
  - strapi3: Uses Node 14 via wrapper script, fixed graphql/knex dependencies
  - strapi4: Created database.js from template
  - PM2 startup systemd service enabled (pm2-deploy.service)
  - Process list saved to /home/deploy/.pm2/dump.pm2

- [x] Set up MRTG monitoring (2025-12-17)
  - Migrated from old server to `/opt/mrtg` (tipmaster/mrtg repo)
  - 45 metrics across 11 categories (CPU, memory, disk, network, databases)
  - Fixed scripts for AlmaLinux 9: unix_socket auth, enp6s0 interface, /proc/net/dev
  - Installed: sysstat, iftop
  - Cron: `/etc/cron.d/mrtg` → every 3 minutes
  - Nginx: mrtg.flywheel.bz via `/opt/serverconfig/etc/nginx/sites-enabled/mrtg.flywheel.bz.conf`
  - Output: `/opt/mrtg/output/` (187 PNG, 47 HTML files)

### Later
- [x] ~~Decide on Cockpit: disable or keep~~ → **Disabled** (2025-12-17)
  - Web-based server management GUI - tested, not needed for now
  - Re-enable if needed: `sudo systemctl enable --now cockpit.socket && sudo firewall-cmd --add-service=cockpit --permanent --reload`

---

## Pre-Reboot Checklist (Verified)

| Check | Status |
|-------|--------|
| SSH Port 42109 configured | OK |
| Root authorized_keys (14 keys) | OK |
| Deploy authorized_keys (14 keys) | OK |
| Firewall allows 42109/tcp | OK |
| sshd service enabled | OK |
| PubkeyAuthentication enabled | OK (default) |
| PermitRootLogin key-based | OK (default) |

---

## Reference: Old Server Data

The `/mnt/Data` drive contains the old server's filesystem with:
- Bootstrap scripts: `/mnt/Data/opt/bootstrap/`
- Projects: `/mnt/Data/opt/` (tmapp, strapi*, gsc-*, serverconfig, etc.)
- Note: Bootstrap scripts are outdated (EL7/PHP5.6) - need modernization for AlmaLinux 9

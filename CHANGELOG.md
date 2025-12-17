# Changelog

## 2025-12-17

### Fedora Dev Environment (fue) Package Parity
- Installed vim 9.1 and htop 3.4.1 to match production baseline
- Replaced Valkey 8.1.5 with Redis 8.4.0 (compiled from source) for version parity with production
- Created systemd service for Redis (`/etc/systemd/system/redis.service`)
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

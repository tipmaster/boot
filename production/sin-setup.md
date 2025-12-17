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

### Base Packages
| Package | Version |
|---------|---------|
| git | 2.47.3 |
| vim | 8.2 |
| htop | 3.3.0 |
| make | 4.3 |
| gcc | 11.5.0 |
| node | 22.19.0 |
| npm | 10.9.3 |
| python3 | 3.9.23 |

### Enabled Repositories
- EPEL 9
- CRB (CodeReady Builder)
- Node.js 22 module stream

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

---

## Pending Tasks

### Later
- [ ] Decide on Cockpit: disable or keep
  - Web-based server management GUI
  - Access: https://101.99.89.98:9090 (if enabled)
  - To disable: `systemctl disable --now cockpit.socket`

### Software Installation (after SELinux disabled)
- [ ] Clone repositories to /opt/
- [ ] Set up symlinks per bootstrap pattern

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

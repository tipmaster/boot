# Production Server Setup: sin

## Server Details

| Property | Value |
|----------|-------|
| Hostname | sin |
| IP | 101.99.89.98 |
| SSH Port | 42109 |
| OS | AlmaLinux 9.7 (Moss Jungle Cat) |
| Data Drive | /mnt/Data (216GB, ext4) |

## Connection

```bash
ssh -p 42109 deploy@sin
ssh -p 42109 root@sin
```

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

### 6. Added sin to local hosts file
```bash
sudo sh -c 'echo "101.99.89.98 sin" >> /etc/hosts'
```

## Pending Tasks

- [ ] Decide on Cockpit: disable or firewall port 9090
  - Cockpit is a web-based server management GUI
  - Access: https://101.99.89.98:9090 (if enabled)
  - To disable: `systemctl disable --now cockpit.socket`

## Resources

| Resource | Total | Used | Available |
|----------|-------|------|-----------|
| Memory | 62 GB | ~1 GB | ~61 GB |
| Swap | 15 GB | 0 | 15 GB |
| Disk (/) | 220 GB | 3.6 GB | 216 GB |
| Disk (/mnt/Data) | 216 GB | 149 GB | 57 GB |

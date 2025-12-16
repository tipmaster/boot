# Project-Specific Configuration

## Environment

**Server Access:**
- Server: <server-name>
- SSH: <user>@<server>:<port>
- Project location: `/path/to/project`

**Database:**
- Type: <database-type>
- Connection: <connection-details>

**Website Configuration:**
- Domain: <domain-name>
- Cloudflare API Key: <cloudflare-api-key> (optional, for DNS management)
- Nginx config file: `/etc/nginx/sites-available/<project-name>`
- SSL: Provided by Cloudflare (no server cert management needed)

## Deployment

**Server Deployment Process:**
1. SSH to server: `ssh <user>@<server> -p <port>`
2. Navigate to project: `cd /path/to/project`
3. Pull latest code: `git pull`
4. Update environment variables if needed (e.g., .env file)
5. Deploy nginx configuration if website changes:
   ```bash
   # Deploy nginx config
   sudo cp deploy/nginx/<project-name> /etc/nginx/sites-available/<project-name>
   sudo ln -sf /etc/nginx/sites-available/<project-name> /etc/nginx/sites-enabled/
   sudo nginx -t && sudo systemctl reload nginx
   ```
6. Restart service:
   - Linux (systemd): `systemctl restart <service-name>`
   - Docker: `docker-compose restart` or `docker restart <container>`
   - PM2: `pm2 restart <app-name>`
   - Custom: See project-specific restart procedure
7. Verify: Check logs and health endpoints
8. Test website through browser (confirm Cloudflare routing works)

**Cron Job Management:**
If this project uses cron jobs:

1. **Version Control:**
   - Cron configuration file location in repo: `deploy/<project-name>.cron`
   - Server cron file location: `/etc/cron.d/<project-name>`

2. **Deployment Process:**
   ```bash
   # Deploy cron configuration to server
   sudo cp /path/to/project/deploy/<project-name>.cron /etc/cron.d/<project-name>
   sudo chmod 644 /etc/cron.d/<project-name>

   # Verify deployment
   cat /etc/cron.d/<project-name>

   # Monitor cron execution (update log path as needed)
   tail -f /var/log/<project-name>/cron.log
   ```

3. **Important Rules:**
   - Never use `crontab -e` on the server
   - Never edit `/etc/cron.d/<project-name>` directly
   - Always version changes in git first
   - Deploy by copying from repo to server

**Health Checks & Monitoring:**
- Health endpoint: `http://<server>/api/health` (replace with actual endpoint)
- Website access: `https://<domain-name>` (via Cloudflare)
- Log locations: `/var/log/<project-name>/` or project-specific log directory
- Service status check:
  - Linux (systemd): `systemctl status <service-name>`
  - Docker: `docker ps` or `docker-compose ps`
  - PM2: `pm2 status` or `pm2 logs <app-name>`
- Nginx status: `sudo nginx -t && sudo systemctl status nginx`

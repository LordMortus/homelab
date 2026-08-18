# Scripts

Utility scripts that run on bare-metal hosts (not in containers).

## cloudflare-ddns.sh

Updates a Cloudflare DNS A record to your current public IP. Runs on the host with the stable uptime (a Pi or small always-on box) via cron or systemd timer.

**Setup:**

1. Edit the variables at the top of the script — `auth_email`, `auth_key`, `zone_identifier`, `record_name`
2. Use `auth_method="token"` with a scoped API token (preferred over global key)
3. The Cloudflare API token needs `Zone:DNS:Edit` on your zone

**Cron example (every 5 minutes):**

```
*/5 * * * * /opt/scripts/cloudflare-ddns.sh >> /var/log/cloudflare-ddns.log 2>&1
```

**systemd timer example:**

```ini
# /etc/systemd/system/cloudflare-ddns.service
[Unit]
Description=Cloudflare DDNS updater

[Service]
Type=oneshot
ExecStart=/opt/scripts/cloudflare-ddns.sh
```

```ini
# /etc/systemd/system/cloudflare-ddns.timer
[Unit]
Description=Run Cloudflare DDNS updater every 5 minutes

[Timer]
OnBootSec=1min
OnUnitActiveSec=5min

[Install]
WantedBy=timers.target
```

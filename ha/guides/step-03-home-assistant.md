# Step 03 — Home Assistant

Add Home Assistant to the existing Docker Compose stack and finish first-time setup.

| | |
|---|---|
| **Prerequisites** | [Step 02](step-02-docker-compose.md) complete |
| **Time** | ~30 minutes |
| **Reference** | [Home Assistant Container install](https://www.home-assistant.io/installation/linux#install-home-assistant-container) |

---

## Goal

- Home Assistant running in Docker alongside Uptime Kuma
- Web UI at `http://<LAN-IP>:8123`
- Config persisted under `/opt/homelab/homeassistant/config`
- Survives reboot

---

## Why `network_mode: host`

On Linux, Home Assistant uses **host networking** so device discovery (mDNS, some integrations) works reliably on a single-NIC mini PC. This is the approach Home Assistant documents for container installs on Linux.

---

## 1. Add Home Assistant to `compose.yml`

```bash
cd /opt/homelab
nano compose.yml
```

Full file (Uptime Kuma + Home Assistant):

```yaml
services:
  uptime-kuma:
    image: louislam/uptime-kuma:1
    container_name: uptime-kuma
    restart: unless-stopped
    volumes:
      - ./uptime-kuma/data:/app/data
    ports:
      - "3001:3001"

  homeassistant:
    image: ghcr.io/home-assistant/home-assistant:stable
    container_name: homeassistant
    restart: unless-stopped
    privileged: true
    network_mode: host
    volumes:
      - ./homeassistant/config:/config
      - /etc/localtime:/etc/localtime:ro
    environment:
      - TZ=Europe/Bucharest
```

Change `TZ` if needed.

---

## 2. Start Home Assistant

```bash
cd /opt/homelab
docker compose pull homeassistant
docker compose up -d homeassistant
docker compose ps
docker compose logs -f homeassistant
```

First start takes **several minutes** (downloads image, creates config). Wait until logs show the core is running, then Ctrl+C.

---

## 3. Open the web UI

Browser: `http://<LAN-IP>:8123`

Examples:
- LAN: `http://192.168.1.50:8123`
- Tailscale: `http://homelab:8123` (if MagicDNS enabled)

---

## 4. Onboarding wizard

1. Create your Home Assistant **owner account** (name, username, password).
2. Set **home location** (used for sun, weather, zone).
3. Choose **metric** units if you prefer.
4. Skip device discovery for now if nothing is plugged in — you can add integrations later.
5. Finish setup — you should land on the default dashboard.

---

## 5. Minimal sanity checks

In Home Assistant:

- **Settings → System → Repairs** — no critical errors
- **Settings → About** — note the Home Assistant version (write it in your rebuild notes)

On the server:

```bash
ls -la /opt/homelab/homeassistant/config/
```

You should see `configuration.yaml`, `.storage/`, etc.

---

## 6. Reboot test

```bash
sudo reboot
```

After reboot:

```bash
docker compose -f /opt/homelab/compose.yml ps
```

Browse to `http://<LAN-IP>:8123` — HA should load and your account should work.

---

## 7. First backup (manual)

Before adding integrations, copy the config folder off the server:

```bash
# On your laptop — replace IP and user
scp -r mario@192.168.1.50:/opt/homelab/homeassistant/config ./ha-backup-$(date +%Y%m%d)
```

Or create a tarball on the server:

```bash
cd /opt/homelab
tar czvf ~/ha-config-backup-$(date +%Y%m%d).tar.gz homeassistant/config
```

Automated backups to another system come in a later guide.

---

## Done when

- [ ] `http://<LAN-IP>:8123` loads Home Assistant
- [ ] Owner account created, dashboard visible
- [ ] `homeassistant` container running after reboot
- [ ] Config exists at `/opt/homelab/homeassistant/config/`
- [ ] At least one manual backup copied elsewhere

---

## Troubleshooting

| Problem | Try |
|---------|-----|
| 8123 not loading | `docker compose logs homeassistant` — wait 5+ min on first boot |
| Permission errors on config | `sudo chown -R mario:mario /opt/homelab/homeassistant` then recreate container |
| Port conflict | Something else on 8123 — `sudo ss -tlnp \| grep 8123` |

---

## What not to do yet

- Do not expose port 8123 to the internet — use Tailscale
- Do not install HACS until core is stable (optional later)
- Zigbee dongle, Pi-hole, MQTT — next guides after Phase 1 planning

---

## Record for rebuild

| Item | Your value |
|------|------------|
| Home Assistant version | |
| LAN URL | http://:8123 |
| Config path | /opt/homelab/homeassistant/config |
| compose.yml path | /opt/homelab/compose.yml |

---

**Next (guide not written yet):** Pi-hole, Mosquitto, Caddy — see [phased roadmap](../vision/home-server-vision.md#phased-roadmap).

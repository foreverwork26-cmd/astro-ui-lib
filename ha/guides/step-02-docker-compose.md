# Step 02 — Docker Compose

Install Docker, create the homelab layout, and verify containers survive a reboot.

| | |
|---|---|
| **Prerequisites** | [Step 01](step-01-os-foundation.md) complete |
| **Time** | ~20–30 minutes |
| **Next guide** | [Step 03 — Home Assistant](step-03-home-assistant.md) |

---

## Goal

- Docker Engine + Compose plugin installed (official packages, not Snap)
- All services will live under `/opt/homelab/`
- One smoke-test container proves the setup works

---

## 1. Install Docker

Run on the server as your normal user (`mario`):

```bash
# Add Docker's official GPG key and repo
sudo apt update
sudo apt install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Add your user to the `docker` group (log out and back in after):

```bash
sudo usermod -aG docker mario
```

**Log out of SSH and log back in** so group membership applies.

Verify:

```bash
docker --version
docker compose version
docker run --rm hello-world
```

---

## 2. Directory layout

```bash
mkdir -p /opt/homelab/{homeassistant/config,uptime-kuma}
cd /opt/homelab
```

Target structure:

```text
/opt/homelab/
  compose.yml       # all services defined here
  .env              # shared variables (optional)
  homeassistant/
    config/         # HA data (step 03)
  uptime-kuma/
    data/           # Uptime Kuma SQLite (this step)
```

---

## 3. Create `compose.yml`

```bash
nano /opt/homelab/compose.yml
```

Paste:

```yaml
# Homelab stack — add services in later guides.
# Docs: https://docs.docker.com/compose/

services:
  uptime-kuma:
    image: louislam/uptime-kuma:1
    container_name: uptime-kuma
    restart: unless-stopped
    volumes:
      - ./uptime-kuma/data:/app/data
    ports:
      - "3001:3001"
```

Smoke test only — Uptime Kuma is also useful in Phase 2 for monitoring.

---

## 4. Start the stack

```bash
cd /opt/homelab
docker compose up -d
docker compose ps
```

Open in browser: `http://<LAN-IP>:3001` (e.g. `http://192.168.1.50:3001`)

Complete the Uptime Kuma wizard (create admin user). You can add monitors later.

---

## 5. Enable Docker at boot

Docker should already start on boot. Confirm:

```bash
sudo systemctl is-enabled docker
```

Expected: `enabled`

---

## 6. Reboot test

```bash
sudo reboot
```

After reboot, SSH back in:

```bash
cd /opt/homelab
docker compose ps
curl -s -o /dev/null -w "%{http_code}" http://127.0.0.1:3001
```

Expected: container `running`, HTTP code `200` or `302`.

---

## Done when

- [ ] `docker compose version` works without `sudo`
- [ ] `/opt/homelab/compose.yml` exists
- [ ] Uptime Kuma loads at `http://<LAN-IP>:3001`
- [ ] After reboot, `docker compose ps` shows `uptime-kuma` running

---

## Troubleshooting

| Problem | Try |
|---------|-----|
| `permission denied` on docker | Log out/in after `usermod -aG docker`; or `newgrp docker` |
| Port 3001 in use | Change `"3002:3001"` in compose and use port 3002 |
| Container not running after reboot | `docker compose logs uptime-kuma` |

---

## Conventions for future guides

- **One** `compose.yml` for the whole homelab — new services get a new `services:` block
- Data in subfolders under `/opt/homelab/` (never inside the image)
- `restart: unless-stopped` on everything that should run 24/7
- Back up `/opt/homelab/` (especially config folders) — covered in a later guide

---

**Next:** [Step 03 — Home Assistant](step-03-home-assistant.md)

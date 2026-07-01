# Step 01 — OS foundation

Install Ubuntu Server, secure basic access, and make the machine reachable on your network.

| | |
|---|---|
| **Hardware** | HP ProDesk 400 G5 (or any x86 mini PC) |
| **OS** | Ubuntu Server **24.04 LTS** (64-bit) |
| **Time** | ~45–60 minutes |
| **Next guide** | [Step 02 — Docker Compose](step-02-docker-compose.md) |

---

## Assumptions (change if yours differ)

- Machine is connected to the router with **Ethernet** (not Wi‑Fi).
- You have another computer on the same network to SSH from.
- You will install **Docker on the host OS** — not Proxmox, not HA OS.
- Hostname: `homelab`
- Linux user: pick one name (examples use `mario` — replace everywhere).

---

## What you need before starting

- [ ] USB flash drive (8 GB+)
- [ ] Another PC to download Ubuntu and flash the USB
- [ ] Ethernet cable to router
- [ ] Monitor + keyboard (only for install; server runs headless after)

---

## 1. Create bootable USB

1. Download **Ubuntu Server 24.04 LTS** ISO: https://ubuntu.com/download/server  
2. Flash with [Rufus](https://rufus.ie/) (Windows) or `dd` / [Balena Etcher](https://etcher.balena.io/) (Mac/Linux).  
3. Plug USB into the HP mini PC.

---

## 2. Install Ubuntu Server

1. Boot from USB (F12 / F9 / Esc at startup — HP usually **F12** for boot menu).
2. Installer choices:
   - Language: your preference
   - **Network:** Ethernet should get DHCP — note the IP shown (you will use it for SSH)
   - **Proxy:** leave empty unless you use one
   - **Mirror:** default
   - **Disk:** **Use entire disk** on the 512 GB NVMe (no LVM needed for a dedicated homelab box; LVM is fine if you prefer)
   - **Profile:**
     - Name: your name
     - Server name: `homelab`
     - Username: `mario` (or yours)
     - Password: strong password
   - **SSH:** ✅ **Install OpenSSH server**
   - **Featured snaps:** skip all (no Docker snap — we install Docker properly in step 02)
3. Reboot when prompted, remove USB.

---

## 3. First login and updates

From the server console (or SSH — see step 4):

```bash
sudo apt update && sudo apt upgrade -y
sudo reboot
```

Log back in after reboot.

---

## 4. SSH from your main computer

Find the IP on the router’s DHCP client list, or on the server:

```bash
ip -4 addr show enp* | grep inet
```

From your laptop (replace IP and user):

```bash
ssh mario@192.168.1.50
```

**Optional — SSH key instead of password** (recommended):

```bash
# On your laptop
ssh-copy-id mario@192.168.1.50
```

From now on, do all work over SSH.

---

## 5. Static IP (pick one method)

### Option A — Router DHCP reservation (easiest)

In the router admin UI, reserve `192.168.1.50` (example) for the HP’s MAC address. No server config change. **Recommended for most homes.**

### Option B — Static IP on the server (netplan)

Edit netplan (interface name may be `enp0s31f6` or `eno1` — check with `ip link`):

```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

Example (adjust IP, gateway, interface to match your LAN):

```yaml
network:
  version: 2
  ethernets:
    enp0s31f6:
      dhcp4: false
      addresses:
        - 192.168.1.50/24
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses:
          - 192.168.1.1
          - 1.1.1.1
```

Apply:

```bash
sudo netplan apply
```

Verify SSH still works before closing the session.

---

## 6. Timezone

```bash
sudo timedatectl set-timezone Europe/Bucharest
timedatectl
```

Use your real timezone if different.

---

## 7. Tailscale (remote access)

```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
```

Follow the URL to authenticate. Note the Tailscale hostname (e.g. `homelab.tail1234.ts.net`).

You can reach the server via Tailscale even when away from home — no port forwarding.

---

## 8. Basic firewall (optional but recommended)

```bash
sudo ufw allow OpenSSH
sudo ufw enable
sudo ufw status
```

Tailscale traffic is handled separately; SSH on LAN still works.

---

## 9. Create homelab directory (prep for step 02)

```bash
sudo mkdir -p /opt/homelab
sudo chown mario:mario /opt/homelab
```

---

## Done when

- [ ] Ubuntu Server 24.04 installed on NVMe
- [ ] `sudo apt update && sudo apt upgrade` completed
- [ ] SSH works from another device: `ssh mario@<LAN-IP>`
- [ ] IP is stable (DHCP reservation or netplan)
- [ ] Tailscale connected: `tailscale status` shows the machine
- [ ] `/opt/homelab` exists and is owned by your user
- [ ] Reboot test: `sudo reboot` — machine comes back, SSH works

---

## Troubleshooting

| Problem | Try |
|---------|-----|
| No network after netplan | Hook up monitor; Ubuntu may use different interface name — `ip link` |
| SSH refused | `sudo systemctl status ssh` — ensure OpenSSH was installed |
| Wrong timezone | `timedatectl list-timezones` then `set-timezone` again |

---

## Record for rebuild (fill in)

| Item | Your value |
|------|------------|
| LAN IP | |
| Username | |
| Hostname | homelab |
| Tailscale name | |
| Router DHCP reservation? | yes / no |

Save this table somewhere safe (password manager notes).

---

**Next:** [Step 02 — Docker Compose](step-02-docker-compose.md)

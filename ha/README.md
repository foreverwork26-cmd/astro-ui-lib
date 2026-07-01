# Home lab documentation

Rebuild the home server from zero using these files. **Vision** = what and why. **Guides** = how, step by step.

## Folder layout

| Path | Purpose |
|------|---------|
| [`vision/home-server-vision.md`](vision/home-server-vision.md) | Goals, architecture, hardware decision, service catalog, phased roadmap |
| [`guides/`](guides/) | Install guides (SOPs) — follow in order |

## Progress

Work through guides in numeric order. Each guide ends with a **Done when** checklist.

| Step | Guide | Status |
|------|-------|--------|
| 01 | [OS foundation](guides/step-01-os-foundation.md) | Not started |
| 02 | [Docker Compose](guides/step-02-docker-compose.md) | Not started |
| 03 | [Home Assistant](guides/step-03-home-assistant.md) | Not started |

Update the Status column as you complete steps (or track in git commits).

## Stack decision

**Docker Compose on Ubuntu Server LTS** (bare metal). Not Proxmox — see [vision doc](vision/home-server-vision.md#decisions).

## Hardware (chosen)

HP ProDesk 400 G5 — i5-9500T, 16 GB RAM, 512 GB M.2 NVMe. Details in [Hardware Conclusion](vision/home-server-vision.md#hardware-conclusion).

## After step 03

Next guides (not written yet): Pi-hole, Mosquitto, Caddy, Tailscale hardening, backups, Zigbee. See phased roadmap in the vision doc.

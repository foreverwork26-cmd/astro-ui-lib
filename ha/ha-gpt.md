# Home Server Vision

> **Purpose:** Build a reliable, low-power home server that acts as the central hub for home automation, family services and a lightweight local AI assistant.
>
> **Out of scope:** NAS, heavy AI workloads, media transcoding, virtualization labs.
>
> **Local AI (in scope):** Small models (roughly 3B–7B), quantized, used for home Q&A, tool-calling, and orchestration — not for long chat sessions or heavy reasoning.
>
> **Local AI (out of scope):** Model training, fine-tuning, large models (13B+), dedicated GPU, always-on high-throughput inference.
>
> **Expectation:** Most intelligence should come from calling Home Assistant and other APIs; the LLM is a thin routing and language layer. More RAM lets a local model fit alongside Docker services; CPU inference will still be slow without a GPU. Upgrade RAM on the chosen platform (up to 64 GB) before buying a second machine for speed.

---

# Goals

The server should:

- Run 24/7 with low power consumption
- Be reliable and easy to maintain
- Be modular so new services can be added over time
- Prefer self-hosted and local-first solutions whenever possible
- Provide a single entry point for everything related to the house

---

# Architecture Principles

- **Home Assistant is the source of truth** for device state, automations, and presence.
- **MQTT** (Mosquitto) is the primary integration bus between services today; a dedicated event bus (e.g. NATS) is a later optimization.
- **The Home Portal is a thin client** — aggregates data from Home Assistant, calendar, and wiki; it does not duplicate automation logic or device control.
- **Phase deployments** — not everything ships at once; see roadmap below.
- **One machine** — the chosen HP ProDesk 400 G5 can run the full stack. Upgrade RAM to 32–64 GB if running local AI on the same host. A second machine is only for faster inference, not because RAM is capped.

---

# Phased Roadmap

## Phase 0 — Foundation

- Hardware, UPS, OS, Docker Compose, backups to external target, Tailscale

## Phase 1 — Core

- Home Assistant, MQTT (Mosquitto), reverse proxy (Caddy), Pi-hole, Zigbee coordinator + ZHA

## Phase 2 — Observability & safety

- Uptime Kuma, Grafana + InfluxDB, NUT (UPS), notifications via Home Assistant mobile app + Apprise

## Phase 3 — Cameras & family data

- go2rtc (camera streaming), CalDAV calendar sync, wiki (BookStack or Wiki.js)

## Phase 4 — Portal & AI

- Thin custom dashboard (or enhanced HA dashboards), local AI agent with Home Assistant tools, household knowledge base search

---

# Functional Areas

## 1. Home Automation

### Core

- Home Assistant
- Automations
- Scenes
- Dashboards
- Notifications

### Radio / protocol strategy

| Protocol | Role | Notes |
|----------|------|-------|
| **Zigbee** | Primary for sensors, switches, plugs | Requires a USB coordinator (e.g. Sonoff ZBDongle-E, ConBee III) |
| **Wi-Fi** | Cameras, some smart devices, ESPHome nodes | Higher power use; fine for mains-powered devices |
| **Thread / Matter** | Future devices | May need a Thread border router; Matter support in Home Assistant is improving |
| **Z-Wave** | Optional | Only if specific devices require it; needs a Z-Wave USB stick |

**Recommendation:** Standardize on **Zigbee for battery-powered sensors** and **Wi-Fi/ESPHome** for custom or mains-powered devices. Add Z-Wave only if a specific product requires it.

### Device integration

| Layer | Choice | Why |
|-------|--------|-----|
| Zigbee | **ZHA** (Zigbee Home Automation) | Built into Home Assistant; simplest setup for beginners; excellent community support |
| Wi-Fi DIY | **ESPHome** | First-class Home Assistant integration; easy firmware and device management |
| Cameras | **go2rtc** | Lightweight streaming; integrates well with Home Assistant |
| MQTT devices | **Mosquitto** + Home Assistant MQTT integration | Standard message bus for non-Zigbee devices and automations |

**Alternative:** Zigbee2MQTT instead of ZHA if you specifically want all Zigbee traffic visible on MQTT. Slightly more setup; choose this later only if you outgrow ZHA.

### Sensors

- Door sensors
- Window sensors
- Motion sensors
- Temperature
- Humidity
- CO₂
- Air quality
- Water leak detection (future)
- Smoke detection (future)

### Presence

- Presence detection
- Geofencing
- Family member status

### Monitoring & history

**Short-term (Home Assistant native):**

- Device status, battery levels, recent history, automations
- **Recorder database: PostgreSQL** (not SQLite) — recommended by Home Assistant for 24/7 use; better performance and reliability as data grows

**Long-term (graphs and trends):**

- **InfluxDB** + **Grafana** via the official Home Assistant integration
- Industry-standard retention, better charts, and the most common setup in the Home Assistant community for historical sensor data

**Recommendation:** Use **PostgreSQL for the HA recorder** from day one, and add **InfluxDB + Grafana in Phase 2** when you want richer long-term graphs beyond HA's built-in history cards.

### Future Integrations

- Smart lights
- Smart switches
- Smart plugs
- Thermostat
- Doorbell
- Cameras
- Energy monitoring
- Car integrations

---

# 2. Home Infrastructure Services

## Network

- **Pi-hole** — network-wide ad blocking and tracking protection
- **Local DNS** — Pi-hole serves as the LAN DNS server; add custom local records (e.g. `ha.home`, `grafana.home`) pointing to the home server
- Configure the **router** to hand out Pi-hole as the DNS server for all clients (DHCP stays on the router)
- Dynamic DNS: only if exposing services without Tailscale (not expected initially)

---

## Connectivity

- **Reverse proxy:** Caddy (recommended for simplicity — automatic HTTPS, easy config)
- **HTTPS** on all web services
- **TLS certificates:** internal CA or Let's Encrypt (if using a public hostname)
- **Remote access:** Tailscale (preferred over port-forwarding admin interfaces to the internet)

---

## Messaging

- **MQTT broker:** Mosquitto
- Authenticated access (username/password or certificates)
- Used by Home Assistant, ESPHome, and other integrations

---

## Monitoring

Monitor the server and services:

| Tool | Purpose |
|------|---------|
| **Uptime Kuma** | HTTP/TCP health checks, status page, downtime alerts |
| **Grafana** + **InfluxDB** | Sensor history and system metrics dashboards |
| **NUT** (Network UPS Tools) | UPS integration; graceful shutdown on power loss |

Track:

- CPU, RAM, disk, temperature, network
- Running services and uptime
- Alerts when services fail (via Apprise, Telegram, or Home Assistant notifications)

---

## Backup

Automatic backups for:

- Home Assistant
- Configuration
- Databases
- Dashboard
- AI configuration

Backups should be stored on a separate dedicated system.

Follow a **3-2-1** mindset: 3 copies of important data, on 2 different media, with 1 copy offsite or at least off the primary server.

---

## Notifications

Primary: **Home Assistant mobile app** push notifications.

Extended delivery via **Apprise** (supports Telegram, email, Discord, and many others):

- Mobile push notifications
- Telegram
- Email
- Discord
- Microsoft Teams (optional)

No need for a custom notification service — Apprise covers unified delivery.

---

# 3. Home Portal / Dashboard

A **thin** web application that aggregates information from other services. It does **not** replace Home Assistant — it links to and displays data from HA, calendar, and wiki.

**Phase 4** — start with Home Assistant dashboards in Phase 1; build the custom portal only when daily use patterns are clear.

## Home

- Current weather
- Indoor temperatures
- Humidity
- CO₂
- Air quality
- Open doors/windows
- Active automations
- Energy usage (when energy hardware is integrated)

Data source: Home Assistant API.

---

## Family

- Shared and personal calendars via **CalDAV sync** (Google Calendar, Apple Calendar, or other CalDAV providers)
- Upcoming events and birthdays
- To-do lists and shopping list (via CalDAV tasks or a simple integrated app — not a custom-built system)

**Recommendation:** Sync with existing family calendars (Google/Apple) via CalDAV rather than self-hosting a full calendar server. Less maintenance, good enough for most households.

---

## House

- Sensor overview, device status, automation status, scene controls, historical graphs

Data source: Home Assistant (read-only display; control actions delegate to HA).

---

## Cameras

- Live camera feeds, baby monitor, snapshot previews

Data source: go2rtc streams embedded in the portal or linked from Home Assistant.

---

## AI

- Chat interface, suggestions, daily summary, quick actions

Data source: local AI agent (see §4); runs on the same HP host (upgrade RAM as needed).

---

## Widgets

Pluggable widgets pulling from external services — not reimplemented internally:

- Weather, calendar, todo, notes, cameras, sensor cards, news, traffic, energy usage, custom widgets

---

# 4. Local AI Assistant

The AI should act as a **Home Agent** rather than a general-purpose chatbot.

## Deployment

| Option | When |
|--------|------|
| **Same machine (HP ProDesk 400 G5)** | Default. Start with 16 GB RAM; upgrade to **32 GB** for 7B + Docker, or **64 GB** for maximum headroom. Platform supports up to 64 GB anytime. |
| **Dedicated second machine** | Only if CPU inference speed is unacceptable — not because the HP runs out of RAM. |

**Model size:** 7B is **capable enough** for this use case — most answers come from calling Home Assistant tools (sensor lookups, calendar queries), not from the model's own knowledge.

**Speed reality:** A 7B model on CPU (no GPU) will feel **slow** for long replies. More RAM helps the model **fit in memory**; it does not make inference fast. Accept slow responses, or add a second machine / GPU later for speed.

## Home Knowledge

Examples:

- What's the temperature in the living room?
- Is any window open?
- Who is currently home?
- What's the CO₂ level?
- Show camera status.

Implemented via Home Assistant REST/WebSocket tools — the model calls APIs, it does not guess.

---

## Calendar Assistant

Natural language calendar management via CalDAV integration.

Examples:

> Schedule dinner with my parents next Friday at 19:00.

> Move tomorrow's dentist appointment to Monday.

---

## Todo Assistant

Examples:

> Add milk to the shopping list.

> Remind me to replace the air filter.

---

## Note Taking

Store and organize notes (in wiki or a dedicated notes app).

Examples:

- Home maintenance
- Project notes
- Ideas
- Family information

Search by keywords or natural language.

---

## Internet Assistant

Lightweight internet queries (optional cloud API for these).

Examples:

- Weather
- News
- Exchange rates
- Simple factual questions

No heavy research or complex reasoning required.

---

## Memory

Persistent searchable memory.

Examples:

- Shopping lists
- Maintenance history
- Warranty information
- Previous notes

Maintenance reminders and consumable tracking (e.g. air filters, batteries) can live in the wiki as structured pages or todo items — no separate inventory app needed.

---

## Suggestions

Proactively generate useful suggestions.

Examples:

- CO₂ has been high for several hours.
- Rain is expected tomorrow; one window is still open.
- You have a calendar event in 30 minutes.

---

# 5. Household Knowledge Base

A searchable wiki for everything related to the house.

**Recommended tools:** BookStack or Wiki.js — both well-supported, easy to run in Docker, good search.

Examples:

- Appliance manuals
- Warranty information
- Maintenance procedures
- Paint colors
- Network documentation
- Electrical panel documentation
- Plumbing information
- Emergency procedures
- Wi-Fi information
- Smart home documentation
- Consumables and spare parts (replaces a separate inventory app)

The AI should be able to search and answer questions from this knowledge base.

---

# 6. Automation Playground

A safe environment for experimentation.

Used for:

- Testing automations
- Testing AI prompts
- Scripts
- Integrations
- New services

**Recommendation:** A second Home Assistant instance or a separate Docker Compose profile — isolated from production automations.

---

# 7. Event Bus (Long-term)

Prefer an event-driven architecture.

**Note:** Home Assistant and MQTT already provide event-style integration today. A standalone bus (e.g. NATS, Redis Streams) is only needed when multiple non-HA services must react to the same events without polling Home Assistant.

Examples of events:

- Door opened
- Window opened
- Motion detected
- Someone arrived home
- Baby sleeping
- CO₂ threshold exceeded
- Calendar updated
- Todo completed

Services should react to events instead of tightly coupling to each other.

---

# Non-Functional Requirements

## Reliability

- Stable
- Automatic restart of services
- Recover gracefully after power loss
- UPS with graceful shutdown via NUT

---

## Performance

- Responsive UI
- Fast AI responses — limited by CPU without GPU; upgrade RAM first, accept slower inference, or add GPU/second machine later for speed
- Fast dashboard loading

---

## Scalability

Easy to add:

- New sensors
- New automations
- New dashboards
- New services
- New AI capabilities

---

## Security

- Local-first
- **No admin UIs directly port-forwarded to the internet** — access via Tailscale or reverse proxy with authentication
- HTTPS everywhere (via Caddy)
- Strong authentication on all web services
- **MQTT and Home Assistant APIs authenticated**
- **Secrets not committed to git** — use `.env` files or Docker secrets
- **2FA** on externally reachable services
- Separate IoT/guest Wi-Fi where possible
- Automatic backups

---

## Maintainability

- Containerized services
- Clear folder structure
- Infrastructure as Code when possible
- Easy upgrades
- Easy rollback

---

# Explicitly Out of Scope

These workloads will run on dedicated systems.

- NAS
- File storage
- Media server
- Heavy local AI models (13B+)
- AI training
- GPU-intensive workloads
- Virtualization lab
- Development workstation

---

# High-Level Architecture

```text
                     Home Portal (thin, Phase 4)
                          │
              ┌───────────┴───────────┐
              │                       │
        Home Assistant          Local AI Agent
        (source of truth)         (same host)
              │                       │
      ┌───────┴────────┐        ┌─────┴────────┐
      │                │        │              │
   Zigbee/ZHA     Automations   Tools       CalDAV
   ESPHome            │        │              │
      │                │        │              │
      └────────────────┴────────┴──────────────┘
                       │
                    MQTT (Mosquitto)
                       │
      ┌────────────────┼────────────────┐
      │                │                │
   Pi-hole         Apprise          BookStack
   Caddy           Uptime Kuma       (wiki)
   Tailscale       Grafana
                     Home Server
```

---

# Decisions

| Topic | Decision |
|-------|----------|
| **Hardware** | **HP ProDesk 400 G5** — i5-9500T, 16 GB / 512 GB M.2 NVMe (Interlink) |
| OS | Debian or Ubuntu Server LTS |
| Orchestration | Docker Compose on bare metal |
| DNS / ad blocking | Pi-hole |
| Reverse proxy | Caddy |
| Remote access | Tailscale |
| MQTT | Mosquitto |
| Zigbee | ZHA + USB coordinator |
| HA recorder | PostgreSQL |
| Long-term graphs | InfluxDB + Grafana |
| Calendar | CalDAV sync (Google/Apple) |
| Wiki | BookStack or Wiki.js |
| AI runtime | Ollama + 7B (quantized) on same host; upgrade RAM to 32–64 GB as needed |
| Cameras | go2rtc for viewing; Frigate only if motion detection is needed (adds CPU load) |
| Notifications | Home Assistant app + Apprise |

---

# Future Discussion Topics

- Backup strategy details
- Network topology
- Disaster recovery
- Sensor ecosystem and device shopping list
- Dashboard technology stack for the custom portal
- UPS and Zigbee dongle selection

---

# Hardware Selection

## Budget

- **Target budget:** ~2000 RON (~400 USD)
- **Stretch budget:** ~3000 RON (~600 USD), only if there is a clear and justified benefit in terms of longevity, reliability or performance.

## Overall Philosophy

The goal is **not** to build a powerful workstation, but a reliable 24/7 home server capable of running many lightweight services simultaneously.

Priority order:

1. Reliability
2. Low power consumption
3. Responsiveness
4. Upgradeability
5. Raw performance

---

## Minimum Requirements (filter checklist)

Use this checklist to quickly rule out machines on any site (Interlink, OLX, eBay, etc.).

### Must have

| Requirement | Minimum | Preferred | Why |
|-------------|---------|-----------|-----|
| **CPU** | Intel Core i5 **T-series** (8th–10th gen) or Ryzen 5 **U-series** | **i5-9500T** (chosen) | 6 cores, ~35 W TDP — quiet and efficient for 24/7 |
| **RAM** | **16 GB** DDR4 | 32–64 GB upgrade path | HA + Docker; 32 GB+ if local AI on same host |
| **Storage** | **512 GB SSD** | 512 GB M.2 NVMe | Docker images, databases, logs |
| **Form factor** | Business mini PC | HP ProDesk Mini | Reliability, cooling, Linux support |
| **Max RAM (platform)** | 32 GB | **64 GB** | Required ceiling if local AI runs on same host |
| **Network** | Gigabit Ethernet | — | Enough for HA, cameras, remote access |
| **USB** | 4+ ports | — | Zigbee coordinator, UPS, spare |
| **RAM slots** | 2 slots | — | Upgrade later without replacing all RAM |

### Strong preference

- **NVMe M.2** explicitly stated on the listing
- **64 GB RAM ceiling** if local AI may run on the same host (HP ProDesk 400 G5 qualifies; Dell OptiPlex 5060 Micro caps at 32 GB — rejected for this reason)
- **T-series or U-series** suffix on CPU model (see avoid list below)
- Room in budget for **UPS** and **Zigbee USB dongle** after the PC

### Priority order when tradeoffs exist

When you cannot get everything, prioritize in this order:

1. **T-series CPU** (24/7 suitability beats newer non-T chips)
2. **16 GB RAM** (beats 11th gen with 8 GB)
3. **512 GB SSD** (beats 256 GB + slightly newer CPU)
4. **64 GB RAM ceiling** (beats 32 GB platform cap — rules out Dell OptiPlex 5060 Micro)
5. **CPU generation** (8500T vs 9500T — small impact; 9500T chosen)

**Rule of thumb:** A higher base price does **not** mean a better machine — compare equal configs (16 GB / 512 GB). The Dell OptiPlex 5060 costs more at base because it includes more RAM/SSD, not because it has a newer CPU (it does not).

### Nice to have (not required)

- 1 TB SSD
- 32 GB RAM pre-installed
- Windows Pro (irrelevant if installing Linux)
- Keyboard/mouse included
- 2.5 GbE (only useful if the rest of the network is 2.5G)

---

# Estimated Workload

## CPU

Most services are lightweight.

| Service | Estimated Load |
|----------|---------------:|
| Home Assistant | Very Low |
| Pi-hole | Negligible |
| MQTT Broker | Negligible |
| Reverse Proxy | Low |
| Custom Dashboard | Low |
| Wiki | Negligible |
| Monitoring | Low |
| Backup Jobs | Low |
| Notification Service | Negligible |
| AI Orchestration | Low |
| Local LLM (3B–7B, occasional use) | Moderate |

### Conclusion

Under normal operation the CPU will likely stay between **5–15% utilization**.

The server is expected to run **many services**, not a few CPU-intensive ones.

---

# Memory (RAM)

RAM is expected to become the limiting resource before CPU.

| Service | Estimated RAM |
|----------|--------------:|
| Home Assistant | 0.5–1 GB |
| Dashboard | ~300 MB |
| Reverse Proxy | ~100 MB |
| Pi-hole | ~200 MB |
| MQTT | <100 MB |
| Monitoring | ~300 MB |
| Wiki | ~300 MB |
| Notification Service | ~100 MB |
| Miscellaneous containers | 1–2 GB |
| Local LLM (if on same host) | 4–8 GB |

### Estimated Usage

Without AI on this host:

- ~4 GB

With AI on this host:

- ~10–16 GB

### Recommendation

- **Buy:** 16 GB (configured on Interlink)
- **Upgrade later if needed:** 32 GB for local AI + full Docker stack; up to **64 GB** supported on this platform
- Dell OptiPlex 5060 Micro caps at 32 GB — not chosen despite Interlink offering a 32 GB upgrade

---

# Storage

Since storage-heavy workloads (NAS, media, backups) are hosted elsewhere, storage requirements remain modest.

Expected usage:

- Operating System
- Docker images
- Containers
- Databases
- Home Assistant
- Dashboard
- Wiki
- Logs
- Local AI models (if hosted here)
- Temporary backups

### Recommendation

- **Buy:** 512 GB M.2 NVMe (configured on Interlink)
- NVMe vs SATA matters less than capacity for a 24/7 server; HP listing confirms NVMe

---

# CPU Recommendations

**Chosen:** Intel Core i5-**9500T** (9th gen, 6 cores, 35 W).

Other T-series chips (8500T, 10500T) are fine but offer negligible gain for this workload. CPU generation was **not** a reason to pick the more expensive Dell base config (8500T is older than 9500T).

# Alternative (not chosen)

## Intel N100 Mini PCs

Rejected — 32 GB RAM ceiling on most models, weak CPU for Docker + local AI growth.

# Recommended Configuration (chosen)

| Component | Spec |
|-----------|------|
| **Model** | HP ProDesk 400 G5 Desktop Mini |
| **CPU** | Intel Core i5-9500T |
| **RAM** | 16 GB DDR4 now → upgrade to 32 or 64 GB anytime |
| **Storage** | 512 GB M.2 PCIe NVMe |
| **Max RAM (platform)** | 64 GB |

# Features to Look For

- 2 RAM slots
- NVMe support
- At least 4 USB ports
- BIOS virtualization support
- Reliable cooling
- Easy access for upgrades

Optional:

- USB-C
- Wi-Fi
- Dual Ethernet

---

# What to Avoid

## Hardware

- Raspberry Pi (as primary hub — acceptable only for secondary roles like a remote coordinator)
- Intel Celeron / Pentium
- Very old Intel NUCs (6th–7th Generation)
- HDD-only systems
- eMMC storage
- **Non-T / non-U CPUs** in always-on roles (e.g. i5-9500, i5-11500 at 65 W — hotter, louder, more power than i5-9500T)
- **8 GB RAM** as shipped without a cheap upgrade path to 16 GB
- **256 GB storage** as final config (too tight once Docker and databases grow)
- Consumer mini PCs with no RAM upgrade path (soldered RAM)

## Overpriced for this use case

- Platforms with **32 GB RAM max** if local AI may run on the same host (e.g. Dell OptiPlex 5060 Micro)
- Assuming **higher base price = better machine** — compare equal RAM/SSD configs instead

---

# Budget Recommendations

## Around 2000 RON (~400 USD)

**Chosen config** on Interlink:

- [HP ProDesk 400 G5](https://www.interlink.ro/calculator-hp-prodesk-400-g5-mini-pc-intel-core-i5-9-refurbished-p76602) — 16 GB + 512 GB M.2 NVMe → **~1,530 RON**
- Remaining budget → UPS + Zigbee dongle

---

## Around 3000 RON

Not needed for the chosen HP configuration. Reserve budget for UPS, Zigbee dongle, and optional RAM upgrade sticks instead of a pricier PC SKU.

---

# Hardware Conclusion

## Chosen machine

**[HP ProDesk 400 G5 — i5-9500T](https://www.interlink.ro/calculator-hp-prodesk-400-g5-mini-pc-intel-core-i5-9-refurbished-p76602)** from Interlink.

| | |
|---|---|
| **Order config** | Base + **16 GB RAM** (+250 RON) + **512 GB SSD** (+220 RON) → **~1,530 RON** |
| **CPU** | i5-9500T (9th gen, 6 cores, 35 W) |
| **Storage** | M.2 PCIe NVMe — confirmed on listing and [HP specs](https://support.hp.com/document/c06403257) |
| **RAM now** | 16 GB |
| **RAM later** | Upgrade to **32 GB** (local AI + Docker) or **64 GB** (maximum headroom) — platform supports up to 64 GB via 2× SODIMM |
| **Official docs** | [HP 400 G5 Mini specifications](https://support.hp.com/document/c06403257) |

### Why HP beats Dell (same job, equal config)

Compared at **16 GB / 512 GB** (~1,530 RON HP vs ~1,627 RON Dell base):

| | HP ProDesk 400 G5 | Dell OptiPlex 5060 Micro |
|---|-------------------|--------------------------|
| CPU | **i5-9500T** (9th gen) | i5-8500T (8th gen) |
| Storage | **NVMe confirmed** | NVMe or SATA — unclear on refurb |
| Max RAM | **64 GB** | **32 GB** (hard ceiling) |
| Equal config price | ~1,530 RON | ~1,627 RON |

The Dell base looks more expensive (~567 RON vs HP base) because it ships **16 GB + 512 GB pre-installed**, not because it is a better platform. It has an **older CPU** and a **lower RAM ceiling**.

**Do not assume more expensive = better.** Compare specs at equal RAM/SSD.

### RAM plan (including local AI on this machine)

| RAM | Use case |
|-----|----------|
| **16 GB** (buy now) | Home Assistant + Docker infra — enough to start |
| **32 GB** (upgrade later) | Add Ollama 7B on the same host |
| **64 GB** (upgrade later) | Full stack + AI with maximum headroom |

Only the HP platform supports the 64 GB path. Dell caps at 32 GB regardless of Interlink upgrades (+721 RON).

More RAM lets the model fit in memory; it does **not** make CPU inference fast. Slow replies are a CPU/GPU limit, not a reason to pick Dell.

### Also buy (separate from PC)

- UPS with USB (NUT)
- Zigbee USB coordinator (e.g. Sonoff ZBDongle-E, ConBee III)

---

## Rejected alternatives

| Machine | Why not chosen |
|---------|----------------|
| [Dell OptiPlex 5060](https://www.interlink.ro/calculator-dell-optiplex-5060-mini-pc-intel-core-i5-refurbished-p68492) | Older CPU (8500T); 32 GB RAM max; NVMe unconfirmed on listing; higher price at equal config without better platform |
| [Lenovo M720q](https://www.interlink.ro/pc-lenovo-thinkcentre-m720q-minipc-intel-core-i5-940-refurbished-p77572) | Older CPU (9400T); 32 GB official max (64 GB unofficial); HP is same price with better CPU and 64 GB official support |
| [HP ProDesk 600 G5 i5-9500](https://www.interlink.ro/calculator-refurbished-hp-prodesk-600-g5-minipc-intel-core-i5-95-second-hand-p76608) | Non-T CPU (65 W) — wrong variant |
| [HP EliteDesk 800 G8](https://www.interlink.ro/calculator-hp-elitedesk-800-g8-mini-pc-intel-core-i5-refurbished-p76200) | Non-T CPU; ~3,028 RON; poor value |

---

# Platform specs & seller questions (HP)

Official source: [HP ProDesk 400 G5 Mini specifications](https://support.hp.com/document/c06403257)

| Question | Answer (official docs) |
|----------|------------------------|
| NVMe or SATA? | **M.2 PCIe NVMe** |
| Second drive slot? | Yes — 2.5" bay (needs HP bracket on M.2-only configs) |
| RAM slots / max? | **2× SODIMM, up to 64 GB** |
| USB ports? | **6** — enough for Zigbee dongle + UPS |
| Wi-Fi included? | Optional on platform — Ethernet is enough |
| VESA mount? | Optional accessory — usually not in box |

**Ask Interlink (HP) before ordering:**

1. Does the +512 GB SSD upgrade **replace** the 256 GB drive or add a second drive?
2. Is RAM **1×8 GB** or **2×4 GB**? (affects cheapest path to 16 GB)
3. Wi-Fi or VESA included in this refurb unit? (only if you care)

---

# Design Principle

If this machine becomes the "brain of the house", **reliability should be prioritized over raw performance**.

Enterprise/business hardware is preferred over consumer mini PCs due to:

- Better build quality
- Proven long-term reliability
- Better cooling
- Easier maintenance
- Greater availability of replacement parts
- Stable BIOS and firmware support

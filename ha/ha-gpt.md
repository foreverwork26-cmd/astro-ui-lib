# Home Server Vision

> **Purpose:** Build a reliable, low-power home server that acts as the central hub for home automation, family services and a lightweight local AI assistant.
>
> **Out of scope:** NAS, heavy AI workloads, media transcoding, virtualization labs.
>
> **Local AI (in scope):** Small models (roughly 3B–7B), quantized, used for home Q&A, tool-calling, and orchestration — not for long chat sessions or heavy reasoning.
>
> **Local AI (out of scope):** Model training, fine-tuning, large models (13B+), dedicated GPU, always-on high-throughput inference.
>
> **Expectation:** Most intelligence should come from calling Home Assistant and other APIs; the LLM is a thin routing and language layer. If local inference feels too slow on the main server, run the AI stack on a **separate dedicated machine** rather than compromising responsiveness of home automation.

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
- **One machine to start** — Docker Compose on a single host is enough; split workloads onto a second machine only when performance or reliability requires it (most likely: local AI).

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

Data source: local AI agent (see §4); runs on the home server or a dedicated AI machine.

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
| **Same machine as Home Assistant** | Small models (3B–4B), occasional short queries, acceptable latency |
| **Dedicated AI machine** | 7B models, frequent use, or when responses must feel snappy |

**Model size:** 7B is **capable enough** for this use case — most answers come from calling Home Assistant tools (sensor lookups, calendar queries), not from the model's own knowledge. The LLM understands natural language and decides which tool to call.

**Speed reality:** A 7B model on a CPU-only home server (no GPU) will feel **slow** for anything beyond short replies — often several seconds to tens of seconds. If that is unacceptable, run Ollama (or similar) on a **separate mini PC** with 32 GB RAM rather than sharing CPU with Home Assistant.

Optional: cloud API fallback for non-sensitive general questions only (weather, news) if local speed is insufficient.

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
- Fast AI responses (may require a dedicated AI machine for local LLM)
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
        (source of truth)    (same host or dedicated)
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

# Decisions (TBD)

| Topic | Current lean |
|-------|----------------|
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
| AI runtime | Ollama + 7B model (quantized); dedicated machine if too slow on main host |
| Cameras | go2rtc for viewing; Frigate only if motion detection is needed (adds CPU load) |
| Notifications | Home Assistant app + Apprise |

---

# Future Discussion Topics

- AI acceleration and dedicated AI machine sizing
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
| **CPU** | Intel Core i5 **T-series** (8th–10th gen) or Ryzen 5 **U-series** | i5-9500T, i5-8500T, i5-10500T | 6 cores, ~35 W TDP — quiet and efficient for 24/7 |
| **RAM** | **16 GB** DDR4 | 32 GB (or 16 GB + upgrade path) | HA + PostgreSQL + Docker stack needs headroom |
| **Storage** | **512 GB SSD** | 512 GB+ NVMe M.2, or 1 TB | Docker images, databases, logs grow over time |
| **Form factor** | Business mini PC | Dell OptiPlex / HP ProDesk / Lenovo ThinkCentre Tiny | Reliability, cooling, Linux support |
| **Network** | Gigabit Ethernet | — | Enough for HA, cameras, remote access |
| **USB** | 4+ ports | — | Zigbee coordinator, UPS, spare |
| **RAM slots** | 2 slots | — | Upgrade later without replacing all RAM |

### Strong preference

- **NVMe M.2** explicitly stated (SATA SSD is acceptable but confirm type)
- **Refurbished business** mini PC with 36-month warranty
- **T-series or U-series** suffix on CPU model (see avoid list below)
- Room in budget for **UPS** and **Zigbee USB dongle** after the PC

### Priority order when tradeoffs exist

When you cannot get everything, prioritize in this order:

1. **T-series CPU** (24/7 suitability beats newer non-T chips)
2. **16 GB RAM** (beats 11th gen with 8 GB)
3. **512 GB SSD** (beats 256 GB + slightly newer CPU)
4. **32 GB RAM or upgrade path** (beats CPU generation bump within 8th–10th gen)
5. **CPU generation** (8500T vs 9500T vs 10500T — smallest impact for this workload)

**Rule of thumb:** i5-8500T + 32 GB beats i5-11500 + 16 GB for this home server. RAM and T-series matter more than generation.

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

- **Minimum:** 16 GB
- **Recommended:** 32 GB (or 16 GB on main host + separate AI machine with 32 GB)

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

- **Minimum:** 512 GB NVMe SSD
- **Recommended:** 1 TB NVMe SSD

Reasons:

- Better endurance
- More free space for future services
- Small price difference
- Better performance

---

# CPU Recommendations

## Recommended CPU Families

### Intel

- Core i5 8th Generation
- Core i5 9th Generation
- Core i5 10th Generation

Examples:

- i5-8500T
- i5-9500T
- i5-10500T

---

### AMD

- Ryzen 5 4000 Series
- Ryzen 5 5000 Series

Examples:

- Ryzen 5 5600U
- Ryzen 5 5650U

---

# Preferred Platform

## Business Mini PCs

Preferred manufacturers:

- Lenovo ThinkCentre Tiny
- HP ProDesk Mini
- Dell OptiPlex Micro

Advantages:

- Enterprise-grade reliability
- Very quiet
- Low power consumption (typically 8–20 W)
- NVMe support
- Two RAM slots
- Excellent Linux compatibility
- Small footprint
- Readily available refurbished

---

# Alternative

## Intel N100 Mini PCs

Advantages:

- Very low power consumption
- Silent
- Brand new hardware
- Affordable

Limitations:

- Lower CPU performance
- Upgradeability depends on the model
- May become limiting as more services are added

Suitable for:

- Home Assistant
- Pi-hole
- Small Docker stacks

Less suitable for:

- Growing ecosystems
- Local AI
- Larger numbers of containers

---

# Recommended Configuration

## CPU

- Intel Core i5-9500T
- Intel Core i5-10500T

Alternative:

- Ryzen 5 5600U

---

## RAM

- **Minimum:** 16 GB DDR4
- **Preferred:** 32 GB DDR4 (or 16 GB on main host + separate AI machine)

---

## Storage

- **Minimum:** 512 GB SSD (NVMe preferred)
- **Preferred:** 1 TB NVMe SSD

---

## Network

- Gigabit Ethernet minimum
- 2.5 GbE preferred (not mandatory)

---

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

- Newer generation (11th gen+) with only 8–16 GB RAM and no 32 GB upgrade option, at premium refurb prices
- Paying for Windows Pro, keyboard, or monitor bundles when the machine will run headless Linux

---

# Budget Recommendations

## Around 2000 RON (~400 USD)

Realistic target on Interlink:

- Intel Core i5 **T-series** (8500T / 9400T / 9500T)
- **16 GB** RAM
- **512 GB** SSD (M.2 NVMe preferred)
- Leaves ~400–500 RON for UPS + Zigbee dongle

32 GB + 1 TB at this price usually requires buying 16 GB now and upgrading RAM/SSD yourself later, or finding a deal on a Dell OptiPlex with seller-installed upgrades.

---

## Around 3000 RON

Consider newer mini PCs such as:

- Beelink
- Minisforum
- GMKtec

With CPUs like:

- Ryzen 5 5600U
- Ryzen 7 5800U
- Recent Intel Core i5 U-series

The higher budget is justified only if:

- Buying new hardware is preferred
- Seller-installed **32 GB RAM** upgrade (e.g. Dell OptiPlex 5060 +721 RON option)
- Longer expected hardware lifespan is important
- A **second mini PC** for dedicated local AI (Ollama + 7B) is included in the budget

---

# Interlink Shortlist (refurbished)

> Researched: Interlink.ro — prices include listed discounts where shown; confirm at checkout.
> Retailer phone: **031.620.73.24** (useful for pre-sale questions below).

## Recommended picks

### 1. Best value — HP ProDesk 400 G5 (primary recommendation)

| | |
|---|---|
| **Link** | [HP ProDesk 400 G5 — i5-9500T](https://www.interlink.ro/calculator-hp-prodesk-400-g5-mini-pc-intel-core-i5-9-refurbished-p76602) |
| **CPU** | Intel Core i5-**9500T** (6 cores, 35 W) |
| **Base** | 8 GB RAM, 256 GB M.2 SSD — **~1,060 RON** |
| **Recommended config** | +16 GB RAM (+250), +512 GB SSD (+220) → **~1,530 RON** |
| **Why** | Best price/performance; explicit M.2; correct T-series CPU; leaves budget for UPS + Zigbee dongle |
| **AI plan** | Main server only; add a second mini PC later for Ollama if 7B is too slow |

### 2. Best upgrade path — Dell OptiPlex 5060

| | |
|---|---|
| **Link** | [Dell OptiPlex 5060 — i5-8500T](https://www.interlink.ro/calculator-dell-optiplex-5060-mini-pc-intel-core-i5-refurbished-p68492) |
| **CPU** | Intel Core i5-**8500T** (6 cores, 35 W) |
| **Base** | **16 GB** RAM, **512 GB** SSD — **~1,627 RON** |
| **Optional upgrades** | +32 GB RAM (+721) → ~2,348 RON; +1 TB SSD (+451) → add to above |
| **Why** | Only Interlink listing with **32 GB RAM upgrade**; strong base config out of the box |
| **Caveat** | Listing says "SSD" not "NVMe" — confirm drive type before buying (see questions below) |

### 3. Best budget alternative — Lenovo ThinkCentre M720q

| | |
|---|---|
| **Link (Win Home)** | [Lenovo M720q — i5-9400T](https://www.interlink.ro/pc-lenovo-thinkcentre-m720q-minipc-intel-core-i5-940-refurbished-p77572) — **~1,150 RON** base |
| **Link (Win Pro)** | [Lenovo M720q — i5-9400T Pro](https://www.interlink.ro/pc-lenovo-thinkcentre-m720q-minipc-intel-core-i5-940-refurbished-p77573) — **~1,348 RON** base |
| **CPU** | Intel Core i5-**9400T** (6 cores, 35 W) |
| **Recommended config** | +16 GB RAM (+250), +512 GB SSD (+220) → **~1,620 RON** (Home) / **~1,818 RON** (Pro) |
| **Why** | Explicit **NVMe**; includes keyboard + mouse; 5-star reviews |
| **Note** | 9400T is slightly older than 9500T — negligible difference for this workload. Prefer Home version if installing Linux (cheaper). |

## Comparison table

| Product | CPU | Base price | Configured (16 GB / 512 GB) | NVMe stated? | 32 GB upgrade? |
|---------|-----|------------|----------------------------|--------------|----------------|
| [HP ProDesk 400 G5](https://www.interlink.ro/calculator-hp-prodesk-400-g5-mini-pc-intel-core-i5-9-refurbished-p76602) | i5-9500T | ~1,060 RON | **~1,530 RON** | Yes (M.2) | No (DIY later) |
| [Dell OptiPlex 5060](https://www.interlink.ro/calculator-dell-optiplex-5060-mini-pc-intel-core-i5-refurbished-p68492) | i5-8500T | ~1,627 RON | **included in base** | Confirm with seller | **Yes (+721 RON)** |
| [Lenovo M720q](https://www.interlink.ro/pc-lenovo-thinkcentre-m720q-minipc-intel-core-i5-940-refurbished-p77572) | i5-9400T | ~1,150 RON | **~1,620 RON** | Yes (NVMe) | No (DIY later) |

## Skip list (Interlink)

| Product | Link | Why skip |
|---------|------|----------|
| HP ProDesk 600 G5 | [i5-9500 (non-T)](https://www.interlink.ro/calculator-refurbished-hp-prodesk-600-g5-minipc-intel-core-i5-95-second-hand-p76608) | **i5-9500** without T — 65 W, not ideal for 24/7; same price band as 400 G5 T-model |
| HP EliteDesk 800 G8 | [i5-11500 (non-T)](https://www.interlink.ro/calculator-hp-elitedesk-800-g8-mini-pc-intel-core-i5-refurbished-p76200) | **~3,028 RON** for 16 GB / 512 GB max; non-T CPU; no 32 GB option — poor value |

## Deployment recommendation

```
Option A (recommended):  HP ProDesk 400 G5 or Lenovo M720q — 16 GB / 512 GB (~1,530–1,620 RON)
                         → main server: Home Assistant + Docker infra
                         → later: second mini PC for Ollama + 7B if needed

Option B (one box longer):  Dell OptiPlex 5060 — 16 GB now, upgrade to 32 GB later (~1,627 → ~2,348 RON)
                         → 32 GB helps RAM pressure; does not fix slow CPU inference for 7B models
```

---

# Questions to Ask Interlink (or any seller)

Call **031.620.73.24** before ordering, especially for the Dell OptiPlex.

### Storage

1. Is the SSD **NVMe M.2** or **2.5" SATA**? (Dell 5060 listing is ambiguous.)
2. Is there a **second drive bay** or free M.2 slot for adding storage later?
3. If ordering an SSD upgrade, does it replace the existing drive or add a second one?

### Memory

4. How many **RAM slots** are populated? How many total (usually 2)?
5. What is the **maximum RAM** supported?
6. For models without a 32 GB upgrade option: can Interlink install **32 GB** on request, or is DIY upgrade with a purchased SO-DIMM the only path?

### Physical / homelab fit

7. How many **USB ports** (front + back)? Need at least one permanently free for Zigbee coordinator; one for UPS if USB-connected.
8. Is **Wi-Fi** included? (Not required if using Ethernet.)
9. Does the unit include a **VESA mount** or bracket?

### Purchase / warranty

10. Can Windows be **removed/replaced with Linux** without affecting warranty?
11. What exactly is covered under the **36-month warranty** for refurbished units?
12. If ordering upgrades (RAM/SSD), are they **installed and tested** before shipping?

---

# Final Recommendation

## Preferred Option

A refurbished business mini PC matching the [filter checklist](#minimum-requirements-filter-checklist) above.

**Interlink shortlist (pick one):**

1. **[HP ProDesk 400 G5 — i5-9500T](https://www.interlink.ro/calculator-hp-prodesk-400-g5-mini-pc-intel-core-i5-9-refurbished-p76602)** — best value; configure 16 GB + 512 GB M.2 (~1,530 RON)
2. **[Dell OptiPlex 5060 — i5-8500T](https://www.interlink.ro/calculator-dell-optiplex-5060-mini-pc-intel-core-i5-refurbished-p68492)** — best if 32 GB upgrade path matters (~1,627 RON base, already 16/512)
3. **[Lenovo M720q — i5-9400T](https://www.interlink.ro/pc-lenovo-thinkcentre-m720q-minipc-intel-core-i5-940-refurbished-p77572)** — solid alternative with explicit NVMe (~1,620 RON configured)

**Target configuration (any seller):**

- **CPU:** Intel Core i5 **T-series** (8500T / 9500T / 10500T) or Ryzen 5 U-series
- **RAM:** **16 GB minimum**, 32 GB preferred (or separate AI machine later)
- **Storage:** **512 GB SSD minimum**, NVMe preferred, 1 TB ideal

**Also budget for (not included in PC price):**

- UPS with USB (NUT integration)
- Zigbee USB coordinator (e.g. Sonoff ZBDongle-E, ConBee III)

This configuration is expected to comfortably run:

- Home Assistant
- 15–30 Docker containers
- Custom dashboard
- Home infrastructure services
- Household wiki
- Monitoring stack
- Notification services

Local AI (Ollama + 7B) should run on a **dedicated second machine** if response speed matters; the main server focuses on reliability for home automation — low power consumption, high reliability, and room for future expansion.

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

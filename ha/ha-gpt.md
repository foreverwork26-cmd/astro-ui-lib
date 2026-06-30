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

# Estimated Workload

## CPU

Most services are lightweight.

| Service | Estimated Load |
|----------|---------------:|
| Home Assistant | Very Low |
| Pi-hole / AdGuard | Negligible |
| MQTT Broker | Negligible |
| Reverse Proxy | Low |
| Custom Dashboard | Low |
| Wiki | Negligible |
| Inventory | Negligible |
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
| Local AI Model | 4–8 GB |

### Estimated Usage

Without AI:

- ~4 GB

With AI:

- ~10–16 GB

### Recommendation

- **Minimum:** 16 GB
- **Recommended:** 32 GB

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
- Local AI models
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

32 GB DDR4

---

## Storage

1 TB NVMe SSD

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

- Raspberry Pi
- Intel Celeron
- Very old Intel NUCs (6th–7th Generation)
- HDD-only systems
- eMMC storage

---

# Budget Recommendations

## Around 2000 RON

Look for refurbished business mini PCs with:

- Intel Core i5 (9th or 10th Generation)
- 32 GB RAM
- 1 TB NVMe SSD

This configuration offers the best value for money.

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
- Lower power consumption is desired
- Longer expected hardware lifespan is important
- Better integrated graphics or newer platform features are beneficial

---

# Final Recommendation

## Preferred Option

A refurbished business mini PC.

Suggested configuration:

- **CPU:** Intel Core i5-9500T or i5-10500T
- **RAM:** 32 GB DDR4
- **Storage:** 1 TB NVMe SSD

This configuration is expected to comfortably run:

- Home Assistant
- 15–30 Docker containers
- Custom dashboard
- Home infrastructure services
- Household wiki
- Monitoring stack
- Notification services
- Lightweight local AI assistant

while maintaining low power consumption, high reliability and room for future expansion.

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

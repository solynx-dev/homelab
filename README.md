# My Homelab Setup

This is my current configuration and setup — a living document that I keep updating as things change. It's very much a work in progress, so thanks for your patience while I build it out over the next little while.

If you spot issues, mistakes, or typos, please let me know via Issues. I'm still fairly new to building repos on GitHub, but I'm learning — so suggestions and help are always welcome. Thank you.

## Navigation
- [Apps](/apps/)
- [Media Server](/media/)
- [Remote Access](/access/)

## How I Got Into Homelabbing

I didn't set out to become a "homelabber" — it kind of just happened.

At first I was just curious. I saw posts online of people building these amazing self-hosted setups: dashboards, media servers, entire networks running from a single machine tucked in a closet — and I was hooked. Not because I needed any of it, really, but because I wanted to understand how it all worked.

I started small. Following YouTube videos with Linux, then learning Docker, then deploying a couple of Docker containers… nothing too serious. But that first successful deployment — watching something I built with my own hands actually work — it hit different. It wasn't just about the tech. It was about full control, learning, and having the freedom to test, break, and rebuild without pressure. That's what kept me hooked.

Homelabbing, for me, became this perfect mix of problem solving and creativity. It's fun. It's frustrating. It's incredibly satisfying. I can spin up apps to try out new ideas, test software in a way that feels safe, and constantly challenge myself to get better at managing systems. It doesn't matter if it's a self-hosted VPN, a personal cloud, or just a private media server — every new service is a little win, a reminder that learning doesn't have to be rigid or formal.

And honestly, it's just cool seeing what other people are building. There's a real sense of community in the homelab world — people showing off their setups, sharing configs, helping each other out. That kind of energy is rare, and it makes the experience even better. I started becoming more active on subreddits like [r/selfhosted](https://www.reddit.com/r/selfhosted/) and [r/homelab](https://www.reddit.com/r/homelab/), where I saw small groups of people (or single devs) building amazing looking apps. I love the creativity.

This repo is a bit of a snapshot of that journey. Some things here are experiments, some are long-term tools I actually use, and some are just here because I wanted to try something new. Either way, it's all part of the process. I'm not trying to build the most impressive setup out there — I'm just building *my* setup, learning as I go.

If you're here just browsing, or thinking about starting your own lab, I hope you find something useful — or at least relatable.

## Core Setup

### Home Server — Lenovo M910q Tiny
- 16GB RAM
- 1TB NVMe — boot drive, Proxmox, and fast storage for the containers that need it
- 256GB Samsung SATA SSD
- 4TB Seagate external USB drive (bulk storage)
- Runs **Proxmox** as the hypervisor

### Reverse Proxy — Hetzner CPX11 (VPS)
- 2 vCPU, 2GB RAM, 40GB storage
- Acts as my **public reverse proxy, strictly for IPv4 ingress**
- Needed because Starlink puts me behind CGNAT, so I have no public IPv4 at home
- Cheap enough to not think about — roughly a music subscription a month

### Spare / Testing — Dell Laptop
- 2 cores
- My old machine, kept around for testing and as a spare node

### High-Performance Testing — ROG Zephyrus G14
- Discrete GPU (GTX 1650), Ryzen 7 5800HS, 40GB RAM, 1TB WD_Black SN850X
- Windows 11 / CachyOS **DUAL BOOT** — for GPU-heavy or resource-intensive experiments

### Management
- **Dockhand** for Docker (started on Portainer, moved over since)
- **Arcane** testing it for docker management since it has an iOS native app.

## Network & Remote Access

- **Internet:** Starlink Residential — CGNAT on IPv4, but native IPv6
- **Overlay:** Tailscale
- **IPv4:** routed in through the Hetzner reverse proxy, which works around CGNAT
- **IPv6:** direct connections straight into the home network — no relay needed

More detail lives in [Remote Access](/access/).

## Self-Hosted Highlights

The media stack and the apps I actually use day to day. The full list with descriptions lives in [Apps](/apps/).

- **Jellyfin** — media server with hardware transcoding (config is in this repo)
- **Immich** — Google Photos replacement
- **Tracktor** — vehicle fuel logging, maintenance logs, and more.
- **Beszel** - server monitoring
- **Uptime Kuma** — status monitoring for all my services


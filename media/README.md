# Self-Hosted Media

This is currently a work in progress. If you have any issues please submit them, or suggest any edits.

# Navigation
- [Apps](/apps/)
- [Media Server](/media/)
  - [Storage and Data Structure](#storage-and-data-structure)
  - [Running the Stack (Compose Profiles)](#running-the-stack-compose-profiles)
  - [Arr Apps](#arr-apps)
    - [Included Applications](#included-applications)
    - [How It Works](#how-it-works)
    - [Features](#features)
  - [Download Clients](#download-clients)
    - [SABnzbd](#sabnzbd)
    - [qBittorrent](#qbittorrent)
  - [qui (qBittorrent Web UI)](#qui-qbittorrent-web-ui)
  - [Gluetun](#gluetun-vpn)
    - [Testing Gluetun Connectivity](#testing-gluetun-connectivity)
    - [Passing Through Containers](#passing-through-containers)
    - [External Container to Gluetun](#external-container-to-gluetun)
    - [Container In Another Docker Compose](#container-in-another-docker-compose)
    - [Arr apps wont connect to prowlarr](#arr-apps-wont-connect-to-prowlarr)
    - [Gluetun Proxmox Fix](#gluetun-proxmox-fix)
    - [Reduce Gluetun Ram Usage](#reduce-gluetun-ram-usage)
    - [Testing Containers](#testing-other-containers)
  - [Additional Services](#additional-services)
    - [Audiobookshelf](#audiobookshelf)
    - [Profilarr](#profilarr)
    - [NewtArr](#newtarr)
  - [iOS Arr Apps](#ios-arr-apps)
  - [Seerr (Media Requests)](#seerr-media-requests)
  - [No VPN Arr Compose](#no-vpn-arr-compose)
  - [Tdarr Automated Media Transcoding](#tdarr-automated-media-transcoding)
- [Remote Access](/access/)


# Storage and Data Structure
My storage is split across two mounts, and every service in the compose file follows the same convention:

- **`/docker`** – per-service configuration and app data. This lives on my **NVMe** for fast storage, so container databases and configs stay quick and responsive. Each service mounts its own subfolder, e.g. `/docker/sonarr:/config`.
- **`/data`** – the media library and downloads. This lives on my **bulk mass storage**, where capacity matters more than speed. It’s mounted into the media apps as `/data`.

Keeping configs on fast storage and media on bulk storage means the arr apps stay snappy while the large media files sit on cheaper, higher-capacity disks. Just as important, downloads and the final library live on the **same `/data` volume**, which lets Sonarr/Radarr use hardlinks and atomic (instant) moves instead of slow copy-and-delete operations.

My folder layout follows the same structure popularized by TechHutTV:
```
data
├── downloads
│   ├── qbittorrent
│   │   ├── completed
│   │   ├── incomplete
│   │   └── torrents
│   └── sabnzbd
│       ├── completed
│       ├── nzb
│       └── tmp
├── movies
├── music
├── shows
└── youtube
```

Create the directory:
```
mkdir -p downloads/qbittorrent/{completed,incomplete,torrents} && mkdir -p downloads/sabnzbd/{completed,nzb,tmp}
```


# Running the Stack (Compose Profiles)
This stack uses **Docker Compose profiles** to make the torrent side optional. Everything tied to torrenting — **Gluetun (VPN)**, **qBittorrent**, **qui**, and **FlareSolverr** — is gated behind the `torrents` profile. Everything else (SABnzbd for Usenet, the arr apps, and the extra services) runs by default.

I personally run **Usenet-only** through SABnzbd, so I leave the torrents profile off. That means Gluetun, qBittorrent, qui, and FlareSolverr never start, and no VPN is needed.

**Run without torrents (my default):**
```
docker compose up -d
```
This starts SABnzbd, Prowlarr, Sonarr, Radarr, Lidarr, Bazarr, and the additional services — no VPN, no torrent client.

**Run with torrents:**
```
docker compose —profile torrents up -d
```
This adds Gluetun, qBittorrent, qui, and FlareSolverr on top, with qBittorrent routed through the Gluetun VPN tunnel.

You can also enable it permanently instead of passing the flag each time, by setting an environment variable in your `.env` file (or exporting it in your shell):
```
COMPOSE_PROFILES=torrents
```

> Because those services carry `profiles: [“torrents”]`, they are **skipped entirely** unless the profile is explicitly enabled. If you never plan to torrent, you can leave the profile off, or delete those service blocks from the compose file altogether.


# Arr apps
The **ARR Stack** is a powerful collection of self-hosted applications designed to automate and organize your entire media library—from movies and TV shows to music and subtitles. This stack simplifies the process of searching, downloading, renaming, organizing, and even subtitle syncing—all while seamlessly integrating with your preferred torrent or Usenet clients.

## Included Applications

| App       | Role                                         | Description |
|————|-———————————————|-————|
| **Radarr** | Movie Management                         | Automates the discovery, downloading, and organization of movies. Supports custom quality profiles, scheduled searches, and renaming logic. |
| **Sonarr** | TV Show Management                       | Tracks your favorite series, monitors for new episodes, downloads automatically, and organizes them with your preferred naming scheme. |
| **Lidarr** | Music Management                         | Designed for managing music collections. Lidarr can monitor artists, albums, and tracks—automating downloads and organizing your music library. |
| **Bazarr** | Subtitle Management                      | Fetches and syncs subtitles for your TV shows and movies using the metadata from Radarr and Sonarr. Supports multiple languages and providers. |
| **Prowlarr** | Indexer Aggregator & Manager         | Acts as a central indexer manager for all other *ARR* apps. Supports over 500 indexers and integrates directly with Radarr, Sonarr, Lidarr, and others for seamless search and filtering. |

## How It Works

1. **Prowlarr** manages your indexers and passes metadata to Radarr/Sonarr/Lidarr.
2. **Radarr**, **Sonarr**, and **Lidarr** monitor for media content you want—whether it’s movies, shows, or music.
3. They communicate with your download clients (like qBittorrent, NZBGet, or SABnzbd) to fetch the files.
4. Once downloaded, the apps automatically rename and move them to the correct folders.
5. **Bazarr** then detects the new media and fetches the best available subtitles for them.
6. The result: a fully organized, ready-to-watch/listen media library—hands-free.

## Features

- Automated downloading with custom quality and language profiles
- Intelligent media organization and renaming
- Subtitles fetched and synced automatically
- Supports both Torrent and Usenet
- API support and full integration between components
- Easily extensible with third-party apps and downloaders


# Download Clients

### SABnzbd
SABnzbd is my **primary downloader** and runs by default (outside the `torrents` profile), since Usenet connects directly over SSL and doesn’t need a VPN. In my opinion it’s much easier to use than NZBGet. I tried NZBGet before and prefer SABnzbd for two reasons. First, it’s visually easier to understand, just like the Arr apps. Second, there’s a nice mobile app, Sable, that looks great and is solid for viewing SABnzbd activity.

### qBittorrent
qBittorrent is **optional** and only starts with the `torrents` profile. When enabled, it routes all of its traffic through Gluetun’s VPN tunnel (`network_mode: service:gluetun`).

**qBittorrent Stalls with VPN Timeout**
I noticed that qBittorrent stalls out if there is a timeout or any type of interruption on the VPN. This is good because it drops the connection, but I need it to start back up when the connection is restored without manually restarting the container.

**Solution #1:** Within the WebUI of qbittorrent head over to advanced options and select `tun0` as the networking interface. See image below.

![qBittorrent GUI Networking Fix](/images/IMG_5813.png)

**Solution #2:** Deunhealth container - automatically restart qbittorrent and prowlarr when they report an unhealthy status.
Add the deunhealth service to your stack. This is already included in arr-compose.yaml
```
  deunhealth:
    image: qmcgaw/deunhealth
    container_name: deunhealth
    network_mode: “none”
    environment:
      - LOG_LEVEL=info
      - HEALTH_SERVER_ADDRESS=127.0.0.1:9999
      - TZ=America/Halifax
    restart: always
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
```
Next add a healthcheck and label to the container(s). Add `deunhealth.restart.on.unhealthy=true` as a label and a simple ping health check as shown below.
```
  qbittorrent:
    image: lscr.io/linuxserver/qbittorrent:latest
    container_name: qbittorrent
    restart: unless-stopped
    labels:
      deunhealth.restart.on.unhealthy=true # Label for deunhealth
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/Halifax
      - WEBUI_PORT=8080
      - TORRENTING_PORT=6881
    volumes:
      - /docker/qbittorrent:/config
      - /data:/data
    network_mode: service:gluetun
    healthcheck:
        test: ping -c 1 www.google.com || exit 1
        interval: 60s
        retries: 3
        start_period: 20s
        timeout: 10s
```
Resources: [DBTech Video on Deunhealth](https://www.youtube.com/watch?v=Oeo-mrtwRgE)


# qui (qBittorrent Web UI)
**qui** is a fast, single-binary qBittorrent web UI from the autobrr team. It’s a modern alternative to qBittorrent’s built-in WebUI (and to Flood, which I used previously), and it’s part of the optional `torrents` profile — so it only starts when torrenting is enabled.

### Features
- Manage one or many qBittorrent instances from a single dashboard
- Fast and responsive even with very large torrent collections
- Cross-seed – automatically find and add matching torrents across trackers
- Rule-based automations for torrent management
- Runs on port `7476`

**qui GitHub [here](https://github.com/autobrr/qui)**


# Gluetun VPN
Gluetun is only used when the `torrents` profile is enabled. It provides the VPN tunnel that qBittorrent routes its traffic through. My config uses WireGuard, tested with Mullvad.

### Testing Gluetun Connectivity
Once your containers are up, you can test your connection to your provider.
```
docker run —rm —network=container:gluetun alpine:3.18 sh -c “apk add wget && wget -qO- https://ipinfo.io”
```

### Passing Through Containers
When containers are in the same docker compose, all you need to add is a network_mode: `service:container_name` and open the ports through the gluetun container. See example from the arr-compose.yaml
```
services:
  gluetun: # This config is for wireguard tested with Mullvad
    image: qmcgaw/gluetun
    container_name: gluetun
    ...
    ports:
      - “8080:8080”
      - “6881:6881”
      - “6881:6881/udp”
  qbittorrent:
    image: lscr.io/linuxserver/qbittorrent:latest
    container_name: qbittorrent
    ...
    network_mode: service:gluetun
```
### External Container to Gluetun
Add `—network=container:gluetun` when launching a container from outside the stack.

### Container in another docker compose
Add `network_mode: “container:gluetun”` to your docker-compose.yaml. Ensure you open the ports through the gluetun container.

### Arr apps wont connect to prowlarr
In my current compose, Prowlarr runs on the `media-net` network rather than through the VPN, so this normally isn’t an issue. But if you **do** route Prowlarr (or qBittorrent) through Gluetun and it can’t reach other services on your LAN, you need to allow Gluetun to access your local subnet. Add the following to your docker compose file under the gluetun environment variables:

`- FIREWALL_OUTBOUND_SUBNETS=192.168.1.0/24 #change to your specific subnet`

### Gluetun Proxmox Fix
“cannot Unix Open TUN device file: operation not permitted and cannot create TUN device file node: operation not permitted” may happen if you’re running this on LXC containers.

Find your container number, mine is 110

edit `/etc/pve/lxc/110.conf` and add:
```
lxc.cgroup2.devices.allow: c 10:200 rwm
lxc.mount.entry: /dev/net dev/net none bind,create=dir
lxc.mount.entry: /dev/net/tun dev/net/tun none bind,create=file
```
Make sure you pass through the tun device (/dev/net/tun:/dev/net/tun) as shown in arr-compose.yaml.

### Reduce Gluetun Ram Usage
Gluetun bundles a recursive caching DNS resolver called unbound for handling domain name requests securely. Over time the cache size, which rests in RAM, can balloon to gigabytes.

You can reduce this by adding the following to your docker compose file under the gluetun environment variables.

`BLOCK_MALICIOUS=off`
This may not be an issue as DNS over HTTPS in Go to replace Unbound is implemented.

### Testing Other Containers
Jump into the Exec console and run the wget command below. Tested with qbittorrent and prowlarr. Ensure you open the ports through the gluetun container.
```
docker exec -it container_name bash
wget -qO- https://ipinfo.io
```


# Additional Services
These services run by default (outside the `torrents` profile) and round out the media stack.

### Audiobookshelf
**Audiobookshelf** is a self-hosted audiobook and podcast server with polished mobile apps. It keeps your listening progress in sync across devices and streams your library from anywhere.

- Audiobook and podcast library management
- Progress sync across devices
- Native iOS and Android apps
- Automatic podcast downloading
- Multi-user support

In my setup it serves on port `1378`, with books stored under `/data/books`.

**Audiobookshelf GitHub [here](https://github.com/advplyr/audiobookshelf)**

### Profilarr
**Profilarr** is a configuration-management tool for Radarr and Sonarr. Instead of manually copying custom formats and quality profiles from TRaSH Guides, Profilarr imports, version-controls, and syncs them across your arr instances.

- Import custom formats and quality profiles from curated databases (TRaSH Guides, Dictionarry)
- Git-based version control of your configurations
- Push/sync configs to any number of Radarr/Sonarr instances
- Test and simulate how a profile scores releases
- Preserves local customizations during updates

Runs on port `6868`.

**Profilarr GitHub [here](https://github.com/Dictionarry-Hub/profilarr)**

### NewtArr
**NewtArr** (by ElfHosted) is a fork of Huntarr v6.6.3 — the last clean release before its later, heavier versions. It continuously hunts your arr libraries for missing content and items that could be upgraded, triggering searches automatically while staying gentle on your indexers.

- Continuously searches Sonarr, Radarr, Lidarr, and more for missing items and quality upgrades
- Rate-limited, indexer-friendly search cycles
- Multi-instance support
- Web UI on port `9705`

> Note: NewtArr ships with `proxy_auth_bypass` enabled by default, which assumes it sits behind an authenticating reverse proxy. If you expose it directly to the internet, make sure you configure authentication.

**NewtArr GitHub [here](https://github.com/elfhosted/newtarr)**


# iOS Arr Apps
Looking to manage your ARR stack from your iPhone? These iOS apps offer varying levels of support, polish, and functionality for remote management and notifications.

| App        | Pricing                         | Highlights                                                                                                                 | Personal Notes                                                   | App Store |
|————|-———————————|-—————————————————————————————————————————|——————————————————————|————|
| **Helmarr** | Free w/ Pro upgrade ($12.99 CAD one-time) | Manage multiple ARR stacks from one dashboard. Customizable and intuitive interface.                                            | Best value for managing **multiple instances**.                 | [![Download on the App Store](https://img.shields.io/badge/App%20Store-Helmarr-blue?logo=apple&style=for-the-badge)](https://apps.apple.com/ca/app/helmarr/id1638624921) |
| **Ruddarr** | Free w/ optional IAPs            | Cleanest visual design, integrates notifications for Sonarr and Radarr.                                             | I prefer Discord for notifications, so I don’t use this one.    | [![Download on the App Store](https://img.shields.io/badge/App%20Store-Ruddarr-blue?logo=apple&style=for-the-badge)](https://apps.apple.com/ca/app/ruddarr/id6476240130) |
| **Sable.** | Free w/ optional IAPs            | Amazing app to view activity on SABnzbd from your iPhone or iPad.                                             | Great app for NZBs.    | [![Download on the App Store](https://img.shields.io/badge/App%20Store-Sable-blue?logo=apple&style=for-the-badge)](https://apps.apple.com/ca/app/sable/id6630387095) |


> ✅ **Ruddarr** + **Sable** are my personal picks.


# Jellyfin Companion Stack
This stack extends Jellyfin’s capabilities with modern interfaces and insightful statistics. It includes multiple apps working in harmony to enhance your media server experience, while still offering API compatibility for third-party dashboards and integrations.

| App             | Role                      | Description                                                                                                                                                   |
|——————|—————————|—————————————————————————————————————————————————————|
| **Seerr**   | Request Management     | The unified successor to Jellyseerr and Overseerr (the two projects merged into Seerr). A media request and discovery manager for Jellyfin, Plex, and Emby. Lets users request movies and TV shows through a sleek web interface, with admin approval workflows and integration with Sonarr/Radarr. Runs on port `5055`. |
| **StreamyStats** | Live Statistics UI     | A visually polished dashboard to view real-time and historical playback statistics from Jellyfin. Preferred for its elegant UI and detailed user insights.    |
| **Jellystat**    | Stats API Backend      | Provides a backend API to expose Jellyfin statistics. While not as visually refined as StreamyStats, it is still vital for supporting external tools.         |

### How They Work Together

- **Seerr** handles user requests and approval queues for your Jellyfin media.
- **StreamyStats** connects to your Jellyfin server to display current viewing sessions, history, bandwidth usage, and user activity in a modern dashboard.
- **Jellystat** silently runs in the background to provide an API layer, ensuring compatibility with Grafana dashboards and other tools that rely on Jellyfin metrics.

> You get the best of both worlds: a beautiful UI from StreamyStats, and broad API support from Jellystat.


# No VPN arr compose
> With the `torrents` profile approach above, you can already run the main stack VPN-free simply by not enabling the profile. This standalone compose is kept as a simpler, self-contained reference if you’d rather run without Gluetun entirely.
```
services:
  qbittorrent:
    image: lscr.io/linuxserver/qbittorrent:latest
    container_name: qbittorrent
    restart: unless-stopped
    ports:
      - 8080:8080 # web interface
      - 6881:6881 # torrent port
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/Halifax
      - WEBUI_PORT=8080
      - TORRENTING_PORT=6881
    volumes:
      - /docker/qbittorrent:/config
      - /data:/data

  nzbget:
    image: lscr.io/linuxserver/nzbget:latest
    container_name: nzbget
    ports:
      - 6789:6789
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/Halifax
      - NZBGET_USER=user
      - NZBGET_PASS=password
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /docker/nzbget:/config
      - /data:/data
    restart: unless-stopped

  prowlarr:
    image: lscr.io/linuxserver/prowlarr:latest
    container_name: prowlarr
    ports:
      - 9696:9696
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/Los_Angeles
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /docker/prowlarr:/config
    restart: unless-stopped

  sonarr:
    image: lscr.io/linuxserver/sonarr:latest
    container_name: sonarr
    restart: unless-stopped
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/Halifax
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /docker/sonarr:/config
      - /data:/data
    ports:
      - 8989:8989

  radarr:
    image: lscr.io/linuxserver/radarr:latest
    container_name: radarr
    restart: unless-stopped
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/Halifax
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /docker/radarr:/config
      - /data:/data
    ports:
      - 7878:7878

  lidarr:
    container_name: lidarr
    image: lscr.io/linuxserver/lidarr:latest
    restart: unless-stopped
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /docker/lidarr:/config
      - /data:/data
    environment:
     - PUID=1000
     - PGID=1000
     - TZ=America/Halifax
    ports:
      - 8686:8686

  bazarr:
    image: lscr.io/linuxserver/bazarr:latest
    container_name: bazarr
    restart: unless-stopped
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/Halifax
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /docker/bazarr:/config
      - /data:/data
    ports:
      - 6767:6767

  flaresolverr:
    # DockerHub mirror flaresolverr/flaresolverr:latest
    image: ghcr.io/flaresolverr/flaresolverr:latest
    container_name: flaresolverr
    environment:
      - LOG_LEVEL=${LOG_LEVEL:-info}
      - LOG_HTML=${LOG_HTML:-false}
      - CAPTCHA_SOLVER=${CAPTCHA_SOLVER:-none}
      - TZ=America/Halifax
    ports:
      - 8191:8191
    restart: unless-stopped
```


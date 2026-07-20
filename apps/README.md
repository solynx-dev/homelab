# Apps

This is frequently updating as I discover new apps. These are mostly my day-to-day **production** apps — the media server lives separately in its own [Media Server](/media/) folder with its own README.

## Navigation
- [Apps](/apps/)
- [Media Server](/media/)
- [Remote Access](/access/)

---

## Infrastructure & Docker Management

### Arcane
A modern, self-hosted Docker management UI for containers, images, and Compose stacks — a clean web dashboard alternative to the CLI.
- Container and Compose stack management
- Image, volume, and network handling
- Real-time resource overview

### Dockhand
A self-hosted Docker management interface focused on real-time container control, visual Compose editing, and multi-host environments.
- Start/stop/restart containers, live logs, and shell access
- Visual Docker Compose stack editor
- Git-based stack deployment with webhooks and auto-sync
- OIDC/SSO login, privacy-focused (zero telemetry)

---

## Networking & Access

### NGINX Proxy Manager
Web GUI for managing an Nginx reverse proxy with automatic Let's Encrypt SSL — handles internal routing and certs behind the Hetzner edge proxy.
- Proxy hosts with one-click SSL
- Let's Encrypt certificate automation
- Access lists and custom locations

### AdGuard Home
Network-wide DNS server that blocks ads and trackers for every device on the network.
- DNS-level ad and tracker blocking
- Custom filtering rules and per-client settings
- Query logging
- Encrypted upstreams (DoH/DoT)

### DDNS
Dynamic DNS updater that keeps my domain records pointed at my current public IP as it changes.
- Automatic DNS record updates
- Multiple provider support (e.g. Cloudflare)
- Built-in health checks

### Pocket ID
A simple self-hosted OIDC provider that uses passkeys for passwordless single sign-on across my services.
- Passkey-only authentication (including YubiKey)
- OIDC/SSO for self-hosted apps
- User groups and access control
- One-time login codes and audit logs

---

## Monitoring & Notifications

### Uptime Kuma
Status and uptime monitoring for all my services, with alerting.
- HTTP/TCP/ping monitors
- Public status pages
- Notifications and response-time history

### Beszel
Lightweight system and server resource monitoring.
- Real-time CPU, RAM, and disk monitoring
- Unlimited machines
- Historical charts and alerts

### Umami
Privacy-friendly, self-hosted web analytics — a Google Analytics alternative.
- Cookieless, no personal data collection
- Simple, clean dashboards
- Multi-site support

### NTFY
Self-hosted push notification service — send alerts to my phone or desktop over simple HTTP.
- Publish via HTTP or CLI
- Topic-based subscriptions
- Mobile and web apps
- Used by other services (like PriceBuddy) for alerts

---

## Documents

### Paperless-NGX
Document management system — scan, OCR, index, and archive documents into a searchable digital archive.
- Full-text OCR search
- Tags, correspondents, and document types
- Automatic scan/mail consumption
- Secure long-term archive

### Paperless-AI
AI companion for Paperless-NGX that auto-tags, titles, and categorizes documents using an LLM (backed by my local Ollama).
- Automatic tagging and classification
- Title generation
- Document analysis and chat
- Uses a local LLM backend

### Scanservjs
Web front-end for my network scanner (SANE) that scans documents straight into the Paperless consume folder.
- Browser-based scanning, no desktop software
- Batch / multi-page scanning
- Adjustable DPI and output format
- Outputs directly into Paperless for automatic import

### BentoPDF
Privacy-first, client-side PDF toolkit (my StirlingPDF replacement) — everything runs locally, nothing gets uploaded.
- Merge, split, and compress PDFs
- Convert to and from other formats
- Sign, annotate, and encrypt
- Built-in OCR

---

## Home & AI

### Home Assistant
Central hub for home automation and smart devices.
- Device integration and automations
- Custom dashboards
- Local control (no cloud dependency)
- HVAC, energy, and presence tracking

### Ollama
Run large language models locally — powers on-device AI for other tools like Paperless-AI.
- Local LLM inference
- Simple model management
- API for other apps to call
- Fully offline, no cloud

---

## Tracking & Utilities

### Sprout Track
Self-hosted tracker for baby activities — feedings, diapers, sleep, and milestones.
- Quick activity logging with a tap-friendly interface
- Daily dashboard and history calendar
- Monthly reports and growth metrics

### Tracktor
Self-hosted vehicle management — track fuel, maintenance, insurance, and documents for my vehicles.
- Fuel logging with efficiency tracking
- Maintenance history per vehicle
- Insurance and document tracking with renewal reminders
- Dashboard with key metrics

### PriceBuddy
Self-hosted price tracker that watches product pages and notifies me when prices drop.
- Track prices across multiple stores
- Price history charts
- Target-price and stock alerts
- Notifications via ntfy, email, and more
- Optional AI-assisted scraping

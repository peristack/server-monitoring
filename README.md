# server-monitoring

A monitoring stack I've built to learn how Prometheus and Grafana work in practice, using a real OracleCloud server I already had running.

## Background

I'm a telecoms engineering student getting into infrastructure. I wanted to work with tools that are able to solve real problems. In addition, I've done some infrastructure research on my own to understand what all the Prometheus + Grafana hype was about beyond just reading docs — so I used an Oracle Cloud VM I already had (which runs a Minecraft server) as the target to monitor. It has ended up being a good excuse to learn Docker Compose properly too.

## What it does

- Collects system metrics (CPU, RAM, disk, network) from the host using Node Exporter
- Collects Minecraft-specific metrics (TPS, MSPT, players online, loaded entities) using FabricExporter + Spark
- Stores everything in Prometheus
- Visualises it all in Grafana with two dashboards

## Stack

- **Prometheus** — scrapes and stores metrics
- **Grafana** — dashboards and visualisation
- **Node Exporter** — system-level metrics
- **FabricExporter** — Minecraft server metrics (requires Spark mod)
- **Docker Compose** — ties it all together
- **Oracle Cloud** — where it runs (Ubuntu 20.04, ARM)

## Architecture
```
Browser
│
└── :3000  →  Grafana
│
Prometheus  ←──  Node Exporter  (system metrics)
│
└──────────  FabricExporter  (Minecraft metrics, via Crafty Controller)
```
Prometheus and Node Exporter communicate internally via Docker networking — no extra ports exposed.

## Setup

### Prerequisites

- Docker and Docker Compose installed
- A Fabric Minecraft server (if you want the Minecraft dashboard)
- Ports 3000 open on your firewall (Grafana UI)

### Run it

\`\`\`bash
git clone https://github.com/peristack/server-monitoring
cd server-monitoring
docker compose up -d
\`\`\`

Grafana will be available at \`http://your-server-ip:3000\`.
Default credentials: \`admin / admin\` — **I recommend changing these immediately and if possible, enabling 2FA or something similar**.

### Minecraft metrics (optional)

Install these mods on your Fabric server:

- [Spark](https://modrinth.com/mod/spark) — performance profiler, provides TPS and MSPT
- [FabricExporter](https://modrinth.com/mod/fabricexporter) — exposes metrics in Prometheus format

FabricExporter listens on port 25585 by default. Update \`prometheus/prometheus.yml\` with your server's address.

### Dashboards

Import these from Grafana's dashboard library:

- **Node Exporter Full** — ID \`1860\` (system metrics)
- **Minecraft Fabric** — ID \`14492\` (game metrics)

## What I learned

- How Prometheus scraping works and how to configure targets
- How Prometheus classifies different types of metrics and why it matters
- How Grafana connects to data sources and how dashboard variables work
- Docker networking between containers (why \`container_name:port\` works internally)
- Basic cloud security — keeping internal services off public ports

## Notes

This is a learning project, not battle-tested for real world use. A few things I'd improve with more time:

- Add Alertmanager for notifications when TPS drops or disk fills up
- Set up proper authentication beyond basic Grafana login
- Use environment variables for secrets instead of hardcoding in compose files

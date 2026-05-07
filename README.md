# server-infra — Observability Stack

Self-hosted observability stack using Loki + Promtail + Grafana. Promtail automatically discovers all Docker containers on the host via Docker socket — no config change needed when adding new apps.

## Stack

| Service | Image | Purpose |
|---|---|---|
| Loki | `grafana/loki:2.9.0` | Log storage & query engine |
| Promtail | `grafana/promtail:2.9.0` | Log collector, scrapes Docker container logs |
| Grafana | `grafana/grafana:10.4.0` | UI, dashboards, alerting |

## Prerequisites

- Docker + Docker Compose v2 installed on the server
- Apps running as Docker containers with labels `app=<name>` and `env=<environment>`

## Setup

**1. Clone this repo on the server:**
```bash
git clone https://github.com/letenk/server-infra
cd server-infra
```

**2. Create `.env` file:**
```bash
cp .env.example .env
```

Edit `.env`:
```
GRAFANA_BIND_IP=<server-ip>
GRAFANA_ADMIN_PASSWORD=<strong-password>
```

**3. Start the stack:**
```bash
make up
```

**4. Open Grafana** at `http://<server-ip>:3000`, login with `admin` / password from `.env`.

## Makefile Commands

```bash
make up        # start all containers (detached)
make down      # stop and remove containers
make restart   # restart all containers
make logs      # follow logs from all containers
make ps        # show container status
make pull      # pull latest images
```

## Adding a New App

Add Docker labels to the app's `docker-compose.yml`:

```yaml
services:
  app:
    labels:
      - "app=my-app-name"
      - "env=production"
    networks:
      - observability-network

networks:
  observability-network:
    external: true
    name: observability-network
```

Promtail auto-discovers the container — no config change needed here.

## Log Retention

Default: **7 days**. Change `retention_period` in `loki/loki-config.yml` if needed.

## Directory Structure

```
.
├── docker-compose.observability.yml
├── Makefile
├── .env.example
├── loki/
│   └── loki-config.yml
├── promtail/
│   └── promtail-config.yml
└── grafana/
    └── provisioning/
        └── datasources/
            └── loki.yml
```

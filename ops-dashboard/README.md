# Ops Dashboard — Grafana + Prometheus + mongodb_exporter

Free, open-source infrastructure monitoring for MongoDB Atlas — the pattern
most shops already run, and the natural pairing for Module 1's failover
demo: watch replication lag and primary/secondary role changes live while
Test Failover runs.

This is deliberately **not** Atlas Charts and **not** a paid Grafana plugin.
`mongodb_exporter` talks to Atlas over the standard MongoDB wire protocol and
exposes metrics for Prometheus to scrape; Grafana reads from Prometheus.
Nothing here needs an Atlas Enterprise/Cloud Grafana license.

## Setup

```bash
cd ops-dashboard
cp .env.example .env   # fill in ATLAS_URI — a user with the clusterMonitor role is enough
docker compose up -d
```

Then:
1. Open **http://localhost:3000** (`admin` / `admin`, change the password on
   first login). Prometheus is already provisioned as a data source.
2. **Dashboards → New → Import**, and load one of these community dashboards
   by ID (both are built specifically for `mongodb_exporter`'s metric names):
   - **11159** — MongoDB Prometheus Exporter Overview
   - **7373** — MongoDB ReplSet Dashboard (replication lag, member state)
3. Run `01_module1_ha_failover.ipynb` and trigger Test Failover — watch the
   primary/secondary roles flip and replication lag spike briefly on the
   ReplSet dashboard while it happens.

## What this covers vs. doesn't

- **Covers well:** replication lag, connections, opcounters, cache
  utilization, replica set member state — infrastructure/ops health. This is
  what answers "teams never suffer undetected replication lag" from the
  original pain-point list.
- **Doesn't cover:** the business-data dashboards from Module 2 (transaction
  volume by status, liquidity). Putting *business* MongoDB queries into
  Grafana needs a MongoDB data source plugin — Grafana Labs' official one is
  Enterprise/Cloud Pro+ only; free community plugins exist but are unsigned
  and less battle-tested. Module 2's notebook covers that ground natively
  instead. See that notebook's intro cell for the tradeoff in more detail.

## Teardown

```bash
docker compose down -v
```

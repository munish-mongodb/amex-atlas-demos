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
   first login). Prometheus is already provisioned as a data source, and a
   **"MongoDB ReplSet — AMEX Ops Dashboard"** is auto-provisioned and shows
   up on the home dashboard list — no manual import needed.
2. Run `01_module1_ha_failover.ipynb` and trigger Test Failover — watch the
   primary/secondary roles flip in the "Replica Set Member State" table and
   replication staleness spike briefly on the timeseries panel while it
   happens.

### Why a hand-built dashboard instead of importing one from grafana.com

Went looking for an existing community dashboard first — every candidate
that looked plausible (IDs 7373, 11159, 12079, 7353, 7359, including two
literally published by Percona's own PMM project) turned out to be built
against an older/different metric-naming generation. `percona/mongodb_exporter`
0.44 exports **raw serverStatus-mirrored names** (`mongodb_ss_connections`,
`mongodb_ss_opcounters`, `mongodb_members_state`, `mongodb_myState`, ...),
not the older curated names (`mongodb_op_counters_total`,
`mongodb_mongod_replset_my_state`, ...) those dashboards expect. They'd have
imported without error and then shown "No data" on every panel — worse than
an obvious failure, since it looks like it worked. So `mongodb-replset.json`
here is hand-built and every panel query was checked directly against
Prometheus (`/api/v1/query`) against this exact cluster before being wired
into the dashboard. If you add panels, use `curl localhost:9216/metrics` to
check the exact metric/label names first rather than assuming a naming
convention from an older exporter version or a different dashboard.

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

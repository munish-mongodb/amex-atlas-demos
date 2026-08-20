# Ops Dashboard — Grafana + Prometheus + mongodb_exporter

Free, open-source infrastructure monitoring for MongoDB Atlas — the pattern
most shops already run, and the natural pairing for Module 1's failover
demo: watch replication lag and primary/secondary role changes live while
Test Failover runs.

`mongodb_exporter` talks to Atlas over the standard MongoDB wire protocol and
exposes metrics for Prometheus to scrape; Grafana reads from Prometheus.
Fully open source end to end — no Atlas Enterprise/Cloud Grafana license
needed.

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

### About `mongodb-replset.json`

`percona/mongodb_exporter` 0.44 exports raw serverStatus-mirrored metric
names (`mongodb_ss_connections`, `mongodb_ss_opcounters`,
`mongodb_members_state`, `mongodb_myState`, ...) rather than the older
curated names (`mongodb_op_counters_total`,
`mongodb_mongod_replset_my_state`, ...) that most published community
dashboards target. So this dashboard is hand-built for this exporter
version, with every panel query checked directly against Prometheus
(`/api/v1/query`) on the live cluster before being wired in. If you add
panels, check exact metric/label names first with
`curl localhost:9216/metrics`.

## Scope

Covers infrastructure/ops health: replication lag, connections, opcounters,
cache utilization, replica set member state. Business-data dashboards
(transaction volume by status, liquidity from Module 2) run natively in
that notebook instead — see its intro cell.

## Teardown

```bash
docker compose down -v
```

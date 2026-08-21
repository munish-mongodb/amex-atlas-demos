# MongoDB Atlas Banking Demos

Eight Jupyter notebooks built around a synthetic banking/money-movement
dataset, covering the MongoDB Atlas capabilities that come up most often in
operational-efficiency conversations: HA and failover, fast analytics,
governed indexing, real-time eventing, vector search, queryable encryption,
multi-region/sharding, and stream processing. All data is synthetic
(Faker-generated or scripted) — nothing here touches production data, and
the whole thing is meant to be left behind for you to run again on your own.

Runs **locally** (venv + `.env`) or in **Google Colab** — every notebook's
first code cell detects which environment it's in and adapts.

## Run in Colab

Click a badge to open a notebook directly in Colab — no local setup needed:

| Lab | Notebook | Colab |
|-----|----------|-------|
| 1 | Setup & Sample Data | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/munish-mongodb/mongodb-atlas-banking-demos/blob/main/01_setup_and_sample_data.ipynb) |
| 2 | HA Failover | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/munish-mongodb/mongodb-atlas-banking-demos/blob/main/02_ha_failover.ipynb) |
| 3 | Operational Analytics | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/munish-mongodb/mongodb-atlas-banking-demos/blob/main/03_operational_analytics.ipynb) |
| 4 | Governed Indexing | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/munish-mongodb/mongodb-atlas-banking-demos/blob/main/04_governed_indexing.ipynb) |
| 5 | Eventing & Vector Search | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/munish-mongodb/mongodb-atlas-banking-demos/blob/main/05_eventing_vector_search.ipynb) |
| 6 | Queryable Encryption | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/munish-mongodb/mongodb-atlas-banking-demos/blob/main/06_queryable_encryption.ipynb) |
| 7 | Multi-Region & Sharding | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/munish-mongodb/mongodb-atlas-banking-demos/blob/main/07_multiregion_and_sharding.ipynb) |
| 8 | Stream Processing | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/munish-mongodb/mongodb-atlas-banking-demos/blob/main/08_stream_processing.ipynb) |

Colab will prompt for your Atlas connection string (nothing is stored in the
notebook or the repo). See **Colab notes** below for the two things that
need an extra step there.

## Run locally

```bash
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env   # fill in ATLAS_URI at minimum
```

Requirements:
- **Atlas cluster, M10 or higher.** Test Failover, Query Insights, and
  Operation Rejection Filters (query blocking) are unavailable on Free/Flex
  tier clusters.
- **MongoDB Server 8.0+** for Lab 4's query-blocking section — Operation
  Rejection Filters are an 8.0 feature. Check your cluster's server version
  in Atlas before running it.
- **A sandbox/demo cluster, not shared production** — Lab 2 triggers a real
  primary failover.
- For Lab 6 (Queryable Encryption): download the Automatic Encryption
  Shared Library (`crypt_shared`) from the MongoDB Download Center and set
  `CRYPT_SHARED_LIB_PATH` in `.env`.
- For the optional Atlas Admin API cells (Lab 2's automated failover, Lab
  7, Lab 8): an Atlas API key pair with **Project Owner** or **Project
  Stream Processing Owner** role, set as `ATLAS_PUBLIC_KEY` /
  `ATLAS_PRIVATE_KEY` / `ATLAS_PROJECT_ID` / `ATLAS_CLUSTER_NAME`.

**Run Lab 1 once before you start** to load sample data. Re-run it any time
to reset to a clean state.

## What each lab covers

| # | Notebook | What it shows |
|---|----------|----------------|
| 1 | `01_setup_and_sample_data.ipynb` | Loads the synthetic customer/transaction data the rest of the labs use |
| 2 | `02_ha_failover.ipynb` | Zero-data-loss failover under live money-movement write load |
| 3 | `03_operational_analytics.ipynb` | Fast aggregation queries and a leadership/Treasury dashboard |
| 4 | `04_governed_indexing.ipynb` | Performance Advisor signal, live query blocking, centralized schema validation |
| 5 | `05_eventing_vector_search.ipynb` | Change Streams (→ Atlas Triggers) and Vector Search for AI readiness |
| 6 | `06_queryable_encryption.ipynb` | Encrypted fields, queryable but unreadable by direct DB access |
| 7 | `07_multiregion_and_sharding.ipynb` | Cluster topology, shard distribution, multi-region notes |
| 8 | `08_stream_processing.ipynb` | Real-time fraud/velocity checks with Atlas Stream Processing |

Labs 1–5 are meant to be run live, in order, in one sitting. Labs 6–8 stand
alone — run them whenever they're relevant, in any order.

## Dashboarding

- **`ops-dashboard/`** — a `mongodb_exporter` → Prometheus → Grafana stack,
  fully open source, for infrastructure metrics (replication lag,
  primary/secondary state, connections). Pairs with Lab 2's failover demo.
  See `ops-dashboard/README.md`.
- **Lab 3's notebook** — the business-data queries (transaction volume,
  liquidity) run natively against MongoDB and chart with matplotlib/pandas —
  no separate BI tool, no data movement, standard aggregation pipeline
  underneath.

## Colab notes

- Every notebook's first code cell detects Colab (`google.colab` in
  `sys.modules`), `pip install`s what it needs, and prompts for your
  `ATLAS_URI` via `getpass` instead of reading `.env`.
- **Lab 6 (Queryable Encryption)**: Colab doesn't have `crypt_shared`
  preinstalled — the notebook downloads the Linux build automatically the
  first time it runs there.
- **Lab 5 (Vector Search)**: installs `sentence-transformers` (and `torch`)
  on first run — expect a couple of minutes the first time, faster after
  Colab caches the packages for that session.
- **Atlas Network Access**: Colab has no fixed IP, so your cluster's IP
  access list needs to allow connections from anywhere (`0.0.0.0/0`) for
  Colab-run notebooks to connect. Use a dedicated sandbox project for this,
  and tighten the access list back down afterward.

## Notes if you're presenting this live

- Lab 2: click **Test Failover** in the Atlas UI while the live plot is
  running. If `ops-dashboard/` is running, have it open on a second screen —
  replication lag and primary/secondary role changes are visible there in
  real time during the election.
- Lab 4 is the one most likely to draw a "doesn't Percona already do this"
  question. Operation Rejection Filters are a core MongoDB Server 8.0
  feature, so Percona Server for MongoDB gets it too once it's on 8.0. The
  differentiator: Query Insights and Performance Advisor surface the bad
  query shape automatically, and Atlas manages that version upgrade rather
  than it being a standalone migration project.
- Lab 5's Vector Search section uses a free local embedding model
  (`sentence-transformers`, no API key) so it runs standalone; mention
  Voyage AI as MongoDB's recommended production embedding provider — same
  index, same query shape, no self-hosted model to manage.
- Lab 6's local master key is written to `customer-master-key.txt` —
  clearly a demo-only stand-in for a real KMS. Say so out loud; don't let
  it look like a recommended production pattern.
- Re-run Lab 1 between a rehearsal and the real session to reset data. Labs
  3 and 5 each seed their own collections independently, so they don't
  strictly depend on Lab 1 having run first.

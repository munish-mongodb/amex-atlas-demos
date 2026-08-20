# AMEX Atlas Demos

MongoDB Atlas demo notebooks built for the AMEX Digital Banking Workshop and
the follow-on operational-efficiency deep dive. Four modules matching the
demo plan, plus two bonus notebooks. All data is synthetic (Faker-generated
or scripted) — nothing here touches real AMEX data, and everything is safe
to leave behind for AMEX engineers to re-run.

Runs **locally** (venv + `.env`) or in **Google Colab** — every notebook's
first code cell detects which environment it's in and adapts.

## Run in Colab

Click a badge, or open any notebook directly:
`https://colab.research.google.com/github/<owner>/<repo>/blob/main/<notebook>.ipynb`

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
- **MongoDB Server 8.0+** for Module 3's query-blocking section — Operation
  Rejection Filters are an 8.0 feature. Check your cluster's server version
  in Atlas before the session.
- **A sandbox/demo cluster, not shared production** — Module 1 triggers a
  real primary failover.
- For the Queryable Encryption bonus notebook: download the Automatic
  Encryption Shared Library (`crypt_shared`) from the MongoDB Download
  Center and set `CRYPT_SHARED_LIB_PATH` in `.env`.
- For the optional Atlas Admin API cells (Module 1's automated failover, the
  multi-region/sharding bonus): an Atlas API key pair with **Project Cluster
  Manager** role, set as `ATLAS_PUBLIC_KEY` / `ATLAS_PRIVATE_KEY` /
  `ATLAS_PROJECT_ID` / `ATLAS_CLUSTER_NAME`.

**Run `00` once before the session** to load sample data. Re-run it any time
to reset to a clean state.

## The four modules

| # | Notebook | Story |
|---|----------|-------|
| 1 | `01_module1_ha_failover.ipynb` | Zero-data-loss failover under live money-movement write load |
| 2 | `02_module2_millisecond_analytics.ipynb` | Fast aggregation queries + leadership/Treasury dashboard |
| 3 | `03_module3_governed_indexing.ipynb` | Performance Advisor signal, live query blocking, centralized schema validation |
| 4 | `04_module4_eventing_vector_search.ipynb` | Change Streams (→ Atlas Triggers) + Vector Search for AI readiness |

Plus two bonus notebooks, not part of the core module flow:

| Notebook | Story |
|----------|-------|
| `05_bonus_queryable_encryption.ipynb` | Encrypted fields, queryable but unreadable by direct DB access — banking insider-threat story |
| `06_bonus_multiregion_and_sharding.ipynb` | Cluster topology inspection, shard distribution, multi-region notes (leave-behind, not presented live) |

## On dashboarding: why not Atlas Charts

The original plan called for an Atlas Charts dashboard in Module 2. That's
swapped out here for two reasons: Charts gets awkward for anything with
multiple linked/cross-filtered panels, and Grafana is the pattern most shops
already run. Two pieces cover that ground instead:

- **`ops-dashboard/`** — a free, open-source `mongodb_exporter` → Prometheus
  → Grafana stack for infrastructure metrics (replication lag,
  primary/secondary state, connections). Pairs with Module 1's failover
  demo. See `ops-dashboard/README.md`.
- **Module 2's notebook** — the business-data queries (transaction volume,
  liquidity) run natively against MongoDB and chart with matplotlib/pandas.
  Putting *that* data into Grafana too needs a MongoDB data source plugin;
  Grafana Labs' official one is Enterprise/Cloud Pro+ only, and free
  community plugins are unsigned and less battle-tested — worth knowing
  before committing to that path for a real dashboard, not something to
  paper over with an unverified live-demo integration.

## Colab notes

- Every notebook's first code cell detects Colab (`google.colab` in
  `sys.modules`), `pip install`s what it needs, and prompts for your
  `ATLAS_URI` via `getpass` instead of reading `.env`.
- **Queryable Encryption bonus notebook**: Colab doesn't have `crypt_shared`
  preinstalled — the notebook downloads the Linux build automatically the
  first time it runs there.
- **Module 4 (Vector Search)**: installs `sentence-transformers` (and
  `torch`) on first run — expect a couple of minutes the first time, faster
  after Colab caches the packages for that session.
- **Atlas Network Access**: Colab has no fixed IP, so your cluster's IP
  access list needs to allow connections from anywhere (`0.0.0.0/0`) for
  Colab-run notebooks to connect. Worth being deliberate about that on a
  cluster used for a banking demo — tighten it back down afterward, or use a
  dedicated sandbox project for this.

## Presenter notes

- Module 1: click **Test Failover** in the Atlas UI while the live plot is
  running — simplest and lowest-risk for a live room. If `ops-dashboard/` is
  running, have it open on a second screen — replication lag and
  primary/secondary role changes are visible there in real time during the
  election.
- Module 3 is the one most likely to draw a "how does this compare to
  Percona today" question — Operation Rejection Filters have no real analog
  in a self-managed MySQL-compatible setup without an external proxy layer
  (ProxySQL rules, etc.).
- Module 4's Vector Search section uses a free local embedding model
  (`sentence-transformers`, no API key) so it runs standalone; mention Voyage
  AI as MongoDB's recommended production embedding provider — same index,
  same query shape, no self-hosted model to manage.
- The Queryable Encryption bonus notebook's local master key is written to
  `customer-master-key.txt` — clearly a demo-only stand-in for a real KMS.
  Say so out loud; don't let it look like a recommended production pattern.
- Re-run notebook `00` between rehearsal and the live session to reset data.
  Module 2 and Module 4 each seed their own collections independently, so
  they don't strictly depend on `00` having run first.

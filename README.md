# SigNoz on Railway

Deploy [SigNoz](https://signoz.io) (open-source observability platform) on [Railway](https://railway.app) using pre-built public images with baked-in configs.

## Architecture

| Service | Base Image | Purpose |
|---------|-----------|---------|
| **clickhouse** | `clickhouse/clickhouse-server:25.5.6` | Time-series storage for traces, metrics, and logs |
| **zookeeper** | `signoz/zookeeper:3.7.1` | Coordination service for ClickHouse |
| **signoz** | `signoz/signoz:v0.117.1` | Query service, API, and web UI |
| **otel-collector** | `signoz/signoz-otel-collector:v0.144.2` | OpenTelemetry data ingestion pipeline |
| **schema-migrator** | `signoz/signoz-otel-collector:v0.144.2` | One-shot database schema setup |

## Railway Setup

### 1. Create a new Railway project

Create five services, each pointing to this repo with a different **Root Directory**:

| Railway Service | Root Directory | Notes |
|----------------|---------------|-------|
| clickhouse | `clickhouse` | Needs a volume mounted at `/var/lib/clickhouse` |
| zookeeper | `zookeeper` | Needs a volume mounted at `/bitnami/zookeeper` |
| signoz | `signoz` | Needs a volume mounted at `/var/lib/signoz` |
| otel-collector | `otel-collector` | Expose ports 4317 (gRPC) and 4318 (HTTP) publicly to receive telemetry |
| schema-migrator | `schema-migrator` | Run once, then stop (or set as a cron job) |

### 2. Configure networking

Railway services can reach each other using their **service name** as the hostname on the private network. The default configs assume the following service names (matching the directory names above):

- `clickhouse` — ClickHouse native TCP on port **9000**, HTTP on port **8123**
- `zookeeper` — ZooKeeper on port **2181**
- `signoz` — Web UI + API on port **8080**
- `otel-collector` — OTLP gRPC on **4317**, OTLP HTTP on **4318**

> **Important:** If your Railway service names differ from the defaults above, override the connection strings via environment variables (see below).

### 3. Deploy order

1. **zookeeper** — start first
2. **clickhouse** — waits for ZooKeeper
3. **schema-migrator** — run after ClickHouse is healthy (creates databases and tables)
4. **signoz** and **otel-collector** — start after migrations complete

### 4. Environment variables

All services have sensible defaults baked in. Override these in Railway's Variables tab if needed:

#### signoz
| Variable | Default | Description |
|----------|---------|-------------|
| `SIGNOZ_TELEMETRYSTORE_CLICKHOUSE_DSN` | `tcp://clickhouse:9000` | ClickHouse connection string |
| `SIGNOZ_SQLSTORE_SQLITE_PATH` | `/var/lib/signoz/signoz.db` | Path to SQLite metadata DB |
| `SIGNOZ_TOKENIZER_JWT_SECRET` | `change-me-to-a-real-secret` | **Change this!** JWT signing secret |
| `SIGNOZ_ALERTMANAGER_PROVIDER` | `signoz` | Alert manager backend |

#### otel-collector / schema-migrator
| Variable | Default | Description |
|----------|---------|-------------|
| `SIGNOZ_OTEL_COLLECTOR_CLICKHOUSE_DSN` | `tcp://clickhouse:9000` | ClickHouse connection string |
| `SIGNOZ_OTEL_COLLECTOR_CLICKHOUSE_CLUSTER` | `cluster` | ClickHouse cluster name |
| `SIGNOZ_OTEL_COLLECTOR_CLICKHOUSE_REPLICATION` | `true` | Enable replication |

#### clickhouse
| Variable | Default | Description |
|----------|---------|-------------|
| `CLICKHOUSE_SKIP_USER_SETUP` | `1` | Skip default user creation (users.xml handles it) |

### 5. Expose the UI

In Railway, add a **public domain** to the `signoz` service on port **8080**. This gives you the SigNoz web dashboard.

To receive telemetry from your applications, expose the `otel-collector` service ports **4317** (gRPC) and/or **4318** (HTTP) publicly.

## Sending telemetry data

Point your OTLP-compatible application to the otel-collector's public URL:

```bash
# gRPC
export OTEL_EXPORTER_OTLP_ENDPOINT=https://your-otel-collector.up.railway.app:443

# HTTP
export OTEL_EXPORTER_OTLP_ENDPOINT=https://your-otel-collector.up.railway.app:443
export OTEL_EXPORTER_OTLP_PROTOCOL=http/protobuf
```

## Updating versions

Edit the `FROM` line in each Dockerfile to pin a newer version. Check [SigNoz releases](https://github.com/SigNoz/signoz/releases) for the latest tags.

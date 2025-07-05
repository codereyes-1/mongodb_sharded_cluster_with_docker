# MongoDB Sharded Cluster with Docker

**Scalable · Fault‑Tolerant · Local Sandbox**

`mongodb_sharded_cluster_with_docker` provides a reproducible Docker Compose setup for a two‑shard MongoDB cluster—each shard a three‑member replica set—plus a three‑member config‑server replica set and a pair of `mongos` routers. The project is aimed at developers who need to explore sharding mechanics or run realistic local integration tests without touching cloud resources.

## Table of Contents

- [Tech Stack](#tech-stack)
- [Installation](#installation)
- [Usage](#usage)
- [License](#license)

## Tech Stack

| Layer             | Components                                                                                    |
| ----------------- | --------------------------------------------------------------------------------------------- |
| **Containers**    | Docker 24 · Docker Compose v2                                                                 |
| **Database**      | MongoDB Community (official `mongo` image)                                                    |
| **Orchestration** | Individual Compose files per cluster role (`config-server/`, `shard1/`, `shard2/`, `mongos/`) |
| **Documentation** | [`00-setup-sharding-doc.md`](00-setup-sharding-doc.md) walkthrough                            |

## Installation

### Prerequisites

- Docker Desktop **24+** (or compatible Docker Engine) with Docker Compose v2
- Unix‑like shell (`bash`, `zsh`, or similar) — Windows PowerShell works as well

### Clone Repository

```bash
git clone https://github.com/codereyes-1/mongodb_sharded_cluster_with_docker.git
cd mongodb_sharded_cluster_with_docker
```

### Bootstrap the Cluster

1. **Config Servers** (replica set `cfgrs`):
   ```bash
   docker compose -f config-server/docker-compose.yaml up -d
   # Initiate replica set
   mongosh mongodb://localhost:40001 < ./config-server/init-config-rs.js   # or follow the manual rs.initiate() in the doc
   ```
2. **Shard 1** (replica set `shard1rs`):
   ```bash
   docker compose -f shard1/docker-compose.yaml up -d
   mongosh mongodb://localhost:50001 < ./shard1/init-shard1-rs.js
   ```
3. **Shard 2** (replica set `shard2rs`):
   ```bash
   docker compose -f shard2/docker-compose.yaml up -d
   mongosh mongodb://localhost:50004 < ./shard2/init-shard2-rs.js
   ```
4. **Router (**``**)**
   ```bash
   docker compose -f mongos/docker-compose.yaml up -d
   ```

> **Tip:** The same steps—including the `rs.initiate()` commands—are documented with inline snippets in [`00-setup-sharding-doc.md`](00-setup-sharding-doc.md).

## Usage

Connect to the cluster via either router:

```bash
# Connect with the MongoDB Shell
mongosh --port 60000  # or 60001 if you add a second router

# Verify sharding status from within mongosh
sh.status()
```

Default port mapping

| Role              | Host Port   |
| ----------------- | ----------- |
| Config Server 0‑2 | 40001‑40003 |
| Shard 1 replicas  | 50001‑50003 |
| Shard 2 replicas  | 50004‑50006 |
| `mongos` Router   | 60000       |

Shards can be added or removed dynamically using `sh.addShard()` after connecting to `mongos`.

## License

Released under the **MIT License**. See the included [`LICENSE`](LICENSE) file for details.


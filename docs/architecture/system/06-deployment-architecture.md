# Deployment Architecture

> Back to [System Architecture](README.md) | [Project README](../../../README.md)

## Table of Contents

- [MVP](#mvp)
  - [Deployment Overview](#deployment-overview)
  - [Deployment Topology](#deployment-topology)
  - [Component Deployment](#component-deployment)
  - [Infrastructure Requirements](#infrastructure-requirements)
  - [Operational Considerations](#operational-considerations)
  - [Independent Deployment Pipelines](#independent-deployment-pipelines)
- [Post-MVP](#post-mvp)
  - [MVP 3: Managed Infrastructure](#mvp-3-managed-infrastructure)
  - [MVP 4: Kubernetes Deployment](#mvp-4-kubernetes-deployment)
  - [MVP 5: Cloud-Native Production](#mvp-5-cloud-native-production)

## MVP

### Deployment Overview

Mnemonic deploys as six containers orchestrated by docker-compose. Three application services handle distinct responsibilities: read-only MCP serving, admin CRUD, and asynchronous enrichment. RabbitMQ decouples write-path operations from enrichment processing. The heavy lifting (LLM inference, tool execution) happens on user workstations via Claude Code.

```mermaid
graph TB
    subgraph "User Workstations"
        WS1[Workstation 1<br/>Claude Code]
        WS2[Workstation 2<br/>Claude Code]
        WS3[Workstation N<br/>Claude Code]
    end

    subgraph "Admin Tools"
        ADMIN[Admin UI / curl]
    end

    subgraph "Server Infrastructure (docker-compose)"
        MCP[mnemonic-mcp<br/>MCP Server :8081]
        API[mnemonic-api<br/>Admin REST API :8080]
        ENRICHER[mnemonic-enricher<br/>Enrichment Worker]
        RMQ[rabbitmq<br/>:5672 / :15672]
        PG[(mnemonic-postgres<br/>PostgreSQL + PGVector<br/>:5432)]
        NEO[(mnemonic-neo4j<br/>Neo4j<br/>:7687)]
    end

    WS1 -->|"MCP"| MCP
    WS2 -->|"MCP"| MCP
    WS3 -->|"MCP"| MCP
    ADMIN -->|"REST"| API
    MCP --> PG
    MCP --> NEO
    API --> PG
    API --> NEO
    API -->|"publish"| RMQ
    RMQ -->|"consume"| ENRICHER
    ENRICHER --> PG
    ENRICHER --> NEO
```

| Container | Image | Port | Role |
|---|---|---|---|
| mnemonic-mcp | ghcr.io/twistingmercury/mnemonic-mcp | 8081 | MCP server (read-only) |
| mnemonic-api | ghcr.io/twistingmercury/mnemonic-api | 8080 | Admin REST API |
| mnemonic-enricher | ghcr.io/twistingmercury/mnemonic-enricher | -- | Enrichment worker |
| rabbitmq | rabbitmq:3-management | 5672/15672 | Message queue |
| mnemonic-postgres | ghcr.io/twistingmercury/mnemonic-postgres | 5432 | PostgreSQL + PGVector |
| mnemonic-neo4j | ghcr.io/twistingmercury/mnemonic-neo4j | 7687 | Neo4j |

### Deployment Topology

#### Logical View

```mermaid
graph TB
    subgraph "Client Tier"
        CC[Claude Code via MCP]
        ADMIN[Admin UI / curl]
    end

    subgraph "Application Tier"
        MCP[mnemonic-mcp<br/>Read-Only MCP Server]
        API[mnemonic-api<br/>Admin REST API]
        ENRICHER[mnemonic-enricher<br/>Enrichment Worker]
    end

    subgraph "Messaging Tier"
        RMQ[RabbitMQ]
    end

    subgraph "Data Tier"
        PG[(Postgres + PGVector)]
        NEO[(Neo4j)]
    end

    CC -->|"MCP"| MCP
    ADMIN -->|"REST"| API
    MCP --> PG
    MCP --> NEO
    API --> PG
    API --> NEO
    API -->|"publish"| RMQ
    RMQ -->|"consume"| ENRICHER
    ENRICHER --> PG
    ENRICHER --> NEO
```

#### Physical View

All six containers run on a single host via docker-compose.

```mermaid
graph TB
    subgraph "Developer Machines"
        D1[Dev 1<br/>Claude Code MCP]
        D2[Dev 2<br/>Claude Code MCP]
        DN[Dev N<br/>Claude Code MCP]
        ADMIN[Admin UI / curl]
    end

    subgraph "Server Environment (docker-compose)"
        MCP[mnemonic-mcp<br/>:8081]
        API[mnemonic-api<br/>:8080]
        ENRICHER[mnemonic-enricher]
        RMQ[rabbitmq<br/>:5672 / :15672]
        PG[(mnemonic-postgres<br/>:5432)]
        NEO[(mnemonic-neo4j<br/>:7687)]
    end

    D1 -->|"MCP"| MCP
    D2 -->|"MCP"| MCP
    DN -->|"MCP"| MCP
    ADMIN -->|"REST"| API
    MCP --> PG
    MCP --> NEO
    API --> PG
    API --> NEO
    API -->|"publish"| RMQ
    RMQ -->|"consume"| ENRICHER
    ENRICHER --> PG
    ENRICHER --> NEO
```

### Component Deployment

#### Mnemonic MCP Server

**Deployment Location:** Server infrastructure (docker-compose)

**Characteristics:**

- Read-only MCP interface for Claude Code pattern searches
- Connects to Postgres (PGVector semantic search) and Neo4j (graph traversal)
- Stateless; horizontally scalable
- No write operations; safe to run multiple instances behind a load balancer

**Resource Requirements:**

| Resource | Expectation |
|---|---|
| CPU | Low (pattern queries, MCP serving) |
| Memory | Moderate (connection pools, response assembly) |
| Storage | None (reads from external databases) |
| Network | Moderate (all MCP client traffic) |

#### Mnemonic Admin API

**Deployment Location:** Server infrastructure (docker-compose)

**Characteristics:**

- REST API for pattern CRUD operations
- Connects to Postgres for persistent storage and RabbitMQ for enrichment dispatch
- Syncs Neo4j best-effort on write operations
- Stateless; horizontally scalable

**Resource Requirements:**

| Resource | Expectation |
|---|---|
| CPU | Low (CRUD operations, message publishing) |
| Memory | Low to moderate (connection pools) |
| Storage | None (writes to external databases) |
| Network | Low (admin traffic only) |

#### Mnemonic Enricher

**Deployment Location:** Server infrastructure (docker-compose)

**Characteristics:**

- Consumes enrichment messages from RabbitMQ
- Connects to Postgres (read/write embeddings), Neo4j (write graph relationships), and OpenAI API (generate embeddings)
- Horizontally scalable; multiple workers can consume from the same queue
- No inbound HTTP traffic

**Resource Requirements:**

| Resource | Expectation |
|---|---|
| CPU | Low to moderate (embedding requests are I/O-bound) |
| Memory | Low to moderate (message processing) |
| Storage | None (writes to external databases) |
| Network | Moderate (OpenAI API calls, database writes) |

#### RabbitMQ

**Deployment Location:** Server infrastructure (docker-compose)

**Characteristics:**

- Message queue for async enrichment dispatch between Admin API and Enricher
- Management UI available on port 15672
- Uses the `rabbitmq:3-management` image (no custom build)

**Resource Requirements:**

| Resource | Expectation |
|---|---|
| CPU | Low (message routing) |
| Memory | Low (small queue depth expected) |
| Storage | Ephemeral (messages are transient; no durability requirement in MVP) |
| Network | Low (internal container traffic only) |

### Infrastructure Requirements

#### Server Infrastructure

The server-side deployment consists of six containers running via docker-compose, suitable for small teams.

```mermaid
graph TB
    subgraph "MVP Deployment (docker-compose)"
        MCP[mnemonic-mcp<br/>:8081]
        API[mnemonic-api<br/>:8080]
        ENRICHER[mnemonic-enricher]
        RMQ[rabbitmq<br/>:5672 / :15672]
        PG[(mnemonic-postgres<br/>:5432)]
        NEO[(mnemonic-neo4j<br/>:7687)]
    end

    MCP --> PG
    MCP --> NEO
    API --> PG
    API --> NEO
    API --> RMQ
    RMQ --> ENRICHER
    ENRICHER --> PG
    ENRICHER --> NEO
```

#### Client Requirements

| Requirement | MVP Local Deployment |
|---|---|
| Claude Code | Required |
| MCP Support | Required |
| Anthropic API key | Via Claude Code |
| Network access | To mnemonic-mcp :8081 |

#### Admin Requirements

| Requirement | MVP Local Deployment |
|---|---|
| HTTP client | Required (Admin UI or curl) |
| Network access | To mnemonic-api :8080 |
| Authentication | None (trusted network) |

### Operational Considerations

#### Monitoring

Key metrics to monitor:

| Component | Metrics |
|---|---|
| mnemonic-mcp | Request rate, latency, error rate, pattern searches |
| mnemonic-api | Write operations, CRUD latency, message publish rate |
| mnemonic-enricher | Queue consumption rate, enrichment latency, OpenAI API errors |
| RabbitMQ | Queue depth, message rates, consumer count |
| Postgres | Connection count, query latency, storage usage |
| Neo4j | Query latency, memory usage, connection count |

#### Logging

All application services emit structured logs with trace correlation via OpenTelemetry (see [Observability Architecture](07-observability-architecture.md)).

| Component | Log Focus |
|---|---|
| mnemonic-mcp | Pattern searches, errors |
| mnemonic-api | CRUD operations, message publishing, errors |
| mnemonic-enricher | Enrichment processing, OpenAI API calls, errors |

### Independent Deployment Pipelines

Each application service and the database images have independent CI/CD pipelines. Database migrations and application code are versioned and deployed independently.

```mermaid
graph TB
    subgraph "Source Repositories"
        MCP_SRC[mnemonic-mcp<br/>src/]
        API_SRC[mnemonic-api<br/>src/]
        ENR_SRC[mnemonic-enricher<br/>src/]
        DBS_SRC[mnemonic-dbs<br/>postgres/ neo4j/]
    end

    subgraph "CI/CD Pipelines"
        MCP_CI[mnemonic-mcp CI/CD<br/>Build, Test, Push to GHCR]
        API_CI[mnemonic-api CI/CD<br/>Build, Test, Push to GHCR]
        ENR_CI[mnemonic-enricher CI/CD<br/>Build, Test, Push to GHCR]
        DBS_CI[mnemonic-dbs CI/CD<br/>Build, Test, Push to GHCR]
    end

    subgraph "Container Registry (GHCR)"
        MCP_IMG[ghcr.io/.../mnemonic-mcp]
        API_IMG[ghcr.io/.../mnemonic-api]
        ENR_IMG[ghcr.io/.../mnemonic-enricher]
        PG_IMG[ghcr.io/.../mnemonic-postgres]
        NEO_IMG[ghcr.io/.../mnemonic-neo4j]
    end

    MCP_SRC --> MCP_CI --> MCP_IMG
    API_SRC --> API_CI --> API_IMG
    ENR_SRC --> ENR_CI --> ENR_IMG
    DBS_SRC --> DBS_CI --> PG_IMG
    DBS_SRC --> DBS_CI --> NEO_IMG
```

Each Go service pipeline follows the standard pattern documented in the [Go CI/CD diagram](../../diagrams/example-go-ci-cd-diagram.md): build Docker image, run quality gates and unit tests in the container, run E2E tests against a compose stack, then push the image to GHCR on merge.

**Pipeline Summary:**

| Pipeline | Repository | Triggers On | Produces |
|---|---|---|---|
| mnemonic-mcp CI/CD | mnemonic-mcp | `src/` changes | MCP server image |
| mnemonic-api CI/CD | mnemonic-api | `src/` changes | Admin API image |
| mnemonic-enricher CI/CD | mnemonic-enricher | `src/` changes | Enricher image |
| mnemonic-dbs CI/CD | mnemonic-dbs | `postgres/` or `neo4j/` changes | Database images |

**Why Independent Pipelines?**

| Scenario | Without Separation | With Separation |
|---|---|---|
| MCP query bug fix | Rebuilds all services | Only mnemonic-mcp deploys |
| New admin endpoint | Rebuilds MCP and enricher too | Only mnemonic-api deploys |
| Schema migration | Rebuilds all application images | Only database images rebuild |
| Enricher tuning | Touches unrelated services | Only mnemonic-enricher deploys |

**Deployment Order for Breaking Changes:**

```text
1. Deploy database images (forward-compatible: nullable/default values)
2. Verify schema changes succeeded
3. Deploy application services (use new schema)
4. (Optional) Deploy tightening migration (add NOT NULL, remove old columns)
```

This separation ensures:

- Faster deployments (only deploy what changed)
- Safer rollbacks (can rollback one service without touching others)
- Clear audit trail (which pipeline changed what)

## Post-MVP

### MVP 3: Managed Infrastructure

Move stateful infrastructure to AWS managed services while keeping application services running locally:

- Replace local PostgreSQL + PGVector with AWS RDS PostgreSQL + PGVector
- Replace local Neo4j with AWS Neptune
- Replace local RabbitMQ with AWS SQS

Application services (mnemonic-mcp, mnemonic-api, mnemonic-enricher) remain local. This validates cloud data-tier connectivity without changing the application runtime.

See [MVP 3 concept diagram](../../diagrams/03-concept-local-iteration.md).

### MVP 4: Kubernetes Deployment

Deploy application services to local Kubernetes (minikube or equivalent single-node cluster):

- Add Envoy proxy as the ingress and authN layer for MCP and Admin API traffic
- Add OPA policies as the authZ checkpoint before requests reach service handlers
- Begin Helm chart development for repeatable local cluster deployment
- Keep data and queue infrastructure on managed AWS services (from MVP 3)

See [MVP 4 concept diagram](../../diagrams/04-concept-local-iteration.md).

### MVP 5: Cloud-Native Production

Deploy the full stack to AWS:

- Move application services from local Kubernetes to AWS Fargate
- Envoy + OPA remain in the request path for authN/authZ
- Provision all AWS infrastructure (networking, compute, managed services) with Terraform
- Use SSM Parameter Store and Secrets Manager for runtime configuration and secrets

See [MVP 5 concept diagram](../../diagrams/05-concept-cloud-iteration.md).

**Next:** [Observability Architecture](07-observability-architecture.md)

---

Copyright (c) 2025-2026 Jeremy K. Johnson. All rights reserved.

%%{init: {
  "theme": "base",
  "themeVariables": {
    "background": "#F3F4F6",
    "primaryTextColor": "#111827",
    "lineColor": "#374151",
    "fontFamily": "Inter, system-ui, sans-serif"
  }
}}%%

flowchart TB

classDef layer fill:#E5E7EB,stroke:#374151,color:#111827,stroke-width:1px;
classDef service fill:#4A90E2,stroke:#1D4ED8,color:#FFFFFF,stroke-width:1.5px;
classDef worker fill:#22C55E,stroke:#15803D,color:#F0FDF4,stroke-width:1.5px;
classDef ui fill:#8B5CF6,stroke:#6D28D9,color:#FFFFFF,stroke-width:1.5px;
classDef redis fill:#EF4444,stroke:#B91C1C,color:#FFFFFF,stroke-width:1.5px;
classDef db fill:#F59E0B,stroke:#B45309,color:#FFFFFF,stroke-width:1.5px;
classDef client fill:#FFFFFF,stroke:#111827,color:#111827,stroke-width:1px;
classDef label fill:#D1D5DB,stroke:#9CA3AF,color:#111827,stroke-width:0.5px;

subgraph CLIENTS["Client Applications"]
direction LR
  APP1["Application 1<br/>OpenTelemetry SDK"]:::client
  APP2["Application 2<br/>Prometheus"]:::client
  APP3["Application 3<br/>Zipkin"]:::client
  APPN["..."]:::client
end

OTLP["OTLP gRPC/HTTP"]:::label
REMOTE["Remote Write"]:::label
ZIPKIN["Zipkin v2"]:::label

subgraph INGEST["Ingestion Layer"]
direction TB
  COLLECTOR["Collector Service<br/>:4317 gRPC, :4318 HTTP<br/>:9090 Prometheus, :9411 Zipkin"]:::service
  BATCHER["Batcher Service<br/>Buffer + Circuit Breaker<br/>10k events capacity"]:::service
  PROCESSOR["Processor Service<br/>Validation → Enrichment<br/>→ Sampling → Storage"]:::service
end

subgraph APPAPI["API & Frontend Layer"]
direction TB
  UI["Web UI<br/>React + TypeScript<br/>Dashboard, Traces, Logs"]:::ui
  CORE["Core API Service<br/>REST API + Auth<br/>Multi-tenant"]:::service
end

subgraph RESILIENCE["Resilience Layer"]
direction TB
  REDISSTREAMS["Redis Streams<br/>Retry Logic + DLQ<br/>Auto-Recovery"]:::redis

  subgraph WK["Workers"]
  direction LR
    WPROC["Worker: Processor"]:::worker
    WCOL["Worker: Collector"]:::worker
    WBAT["Worker: Batcher"]:::worker
  end
end

subgraph STORAGE["Storage Layer"]
direction LR
  REDISCACHE["Redis<br/>Cache"]:::redis
  POSTGRES["PostgreSQL<br/>Users, Tenants<br/>Auth, Alerts"]:::db
  CLICKHOUSE["ClickHouse<br/>Traces, Logs<br/>Metrics"]:::db
end

APP1 --> OTLP
APP2 --> REMOTE
APP3 --> ZIPKIN

OTLP -->|encaminha| COLLECTOR
REMOTE -->|encaminha| COLLECTOR
ZIPKIN -->|encaminha| COLLECTOR

COLLECTOR -->|encaminha| BATCHER
BATCHER -->|processa| PROCESSOR
PROCESSOR -->|conecta| CLICKHOUSE

COLLECTOR -.->|reencaminha| REDISSTREAMS
BATCHER -.->|reencaminha| REDISSTREAMS
PROCESSOR -.->|reencaminha| REDISSTREAMS

REDISSTREAMS -->|reprocessa| WPROC
REDISSTREAMS -->|reprocessa| WCOL
REDISSTREAMS -->|reprocessa| WBAT

WPROC -->|reencaminha| CLICKHOUSE
WCOL -->|reencaminha| BATCHER
WBAT -->|reencaminha| PROCESSOR

UI -->|conecta| CORE
CORE -->|conecta| REDISCACHE
CORE -->|conecta| POSTGRES
CORE -->|conecta| CLICKHOUSE

linkStyle 0 stroke:#374151,stroke-width:1.5px
linkStyle 1 stroke:#374151,stroke-width:1.5px
linkStyle 2 stroke:#374151,stroke-width:1.5px
linkStyle 3 stroke:#111827,stroke-width:2px
linkStyle 4 stroke:#111827,stroke-width:2px
linkStyle 5 stroke:#111827,stroke-width:2px
linkStyle 6 stroke:#111827,stroke-width:2px
linkStyle 7 stroke:#111827,stroke-width:1.5px,stroke-dasharray:4 3
linkStyle 8 stroke:#111827,stroke-width:1.5px,stroke-dasharray:4 3
linkStyle 9 stroke:#111827,stroke-width:1.5px,stroke-dasharray:4 3
linkStyle 10 stroke:#111827,stroke-width:1.5px
linkStyle 11 stroke:#111827,stroke-width:1.5px
linkStyle 12 stroke:#111827,stroke-width:1.5px
linkStyle 13 stroke:#111827,stroke-width:1.5px
linkStyle 14 stroke:#111827,stroke-width:1.5px
linkStyle 15 stroke:#111827,stroke-width:1.5px
linkStyle 16 stroke:#111827,stroke-width:1.5px
linkStyle 17 stroke:#111827,stroke-width:1.5px
linkStyle 18 stroke:#111827,stroke-width:1.5px
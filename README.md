<div align="center">

# ⚡ PULSE - APM

### Enterprise Observability for Modern Infrastructure

*Real-Time Application Flow Map, Distributed Telemetry Ingestion, and Production Observability Platform*

[![Node.js](https://img.shields.io/badge/Node.js-20+-339933?style=flat-square&logo=node.js&logoColor=white)](https://nodejs.org/)
[![React](https://img.shields.io/badge/React-19-61DAFB?style=flat-square&logo=react&logoColor=black)](https://react.dev/)
[![React Flow](https://img.shields.io/badge/React_Flow-xyflow-FF0072?style=flat-square)](https://reactflow.dev/)
[![SQLite](https://img.shields.io/badge/SQLite-WAL_Engine-003B57?style=flat-square&logo=sqlite&logoColor=white)](https://www.sqlite.org/)
[![WebSockets](https://img.shields.io/badge/WebSockets-ws-010101?style=flat-square&logo=socketdotio&logoColor=white)](https://github.com/websockets/ws)
[![NGINX](https://img.shields.io/badge/NGINX-Reverse_Proxy-009639?style=flat-square&logo=nginx&logoColor=white)](https://nginx.org/)
[![PM2](https://img.shields.io/badge/PM2-Cluster_Mode-2B037A?style=flat-square&logo=pm2&logoColor=white)](https://pm2.keymetrics.io/)
[![License](https://img.shields.io/badge/License-Proprietary_Case_Study-blue?style=flat-square)](#-confidentiality--portfolio-notice)

</div>

> [!NOTE]
> **Confidentiality & Engineering Portfolio Notice**  
> This repository serves as an architectural case study and technical reference implementation of **PULSE - APM** (`serverV2`). It demonstrates production-tested patterns for distributed systems observability, zero-code telemetry extraction, real-time graph visualization, and high-throughput ingestion. Proprietary client datasets, internal IP addresses, production security certificates, and private corporate keys are strictly omitted in compliance with security best practices.

---

## 📑 Table of Contents

1. [Executive Summary & Problem Space](#1-executive-summary--problem-space)
2. [Engineering Ownership & Technical Scope](#2-engineering-ownership--technical-scope)
3. [System Architecture & Data Flow](#3-system-architecture--data-flow)
   - [High-Level Distributed Topology](#high-level-distributed-topology)
   - [Telemetry Ingestion & Broadcast Pipeline](#telemetry-ingestion--broadcast-pipeline)
   - [Tiered Storage & Lifecycle Compaction](#tiered-storage--lifecycle-compaction)
4. [Core Platform Capabilities](#4-core-platform-capabilities)
   - [Dynamic Application Flow Map](#dynamic-application-flow-map)
   - [Live Distributed Request Stream Log](#live-distributed-request-stream-log)
   - [Host Infrastructure & Vitals Telemetry](#host-infrastructure--vitals-telemetry)
   - [Automated Root-Cause Isolation & Deadman Liveness](#automated-root-cause-isolation--deadman-liveness)
5. [Key Engineering Challenges & Solutions](#5-key-engineering-challenges--solutions)
   - [Challenge 1: Zero-Code vs. In-Process Telemetry Ingestion](#challenge-1-zero-code-vs-in-process-telemetry-ingestion)
   - [Challenge 2: Real-Time 60 FPS Graph Layout & Metric Edge Animation](#challenge-2-real-time-60-fps-graph-layout--metric-edge-animation)
   - [Challenge 3: High-Throughput Ingestion with SQLite WAL & Automated Compaction](#challenge-3-high-throughput-ingestion-with-sqlite-wal--automated-compaction)
   - [Challenge 4: Cascading Failure Detection & Deadman Liveness Engine](#challenge-4-cascading-failure-detection--deadman-liveness-engine)
   - [Challenge 5: Constant-Time Cryptographic Auth & Multi-Tenant Isolation](#challenge-5-constant-time-cryptographic-auth--multi-tenant-isolation)
6. [Technology Stack & Decision Matrix](#6-technology-stack--decision-matrix)
7. [Production Deployment & Infrastructure Topology](#7-production-deployment--infrastructure-topology)
   - [Target Infrastructure Architecture (3-Server Fleet)](#target-infrastructure-architecture-3-server-fleet)
   - [Central Portal Deployment (NGINX + PM2)](#central-portal-deployment-nginx--pm2)
   - [Zero-Inbound Security Model](#zero-inbound-security-model)
8. [Reliability, SLA Benchmarks & Failure Modes](#8-reliability-sla-benchmarks--failure-modes)
9. [Key Engineering Takeaways & Lessons Learned](#9-key-engineering-takeaways--lessons-learned)
10. [Future Architectural Roadmap](#10-future-architectural-roadmap)
11. [Author & Technical Leadership](#11-author--technical-leadership)

---

## 1. Executive Summary & Problem Space

Modern enterprise architectures span heterogeneous environments: legacy PHP monolithic portals, containerized Node.js microservice clusters, distributed memory caches (Redis), and polyglot persistence tiers (MySQL, MongoDB). In high-throughput, multi-server production environments, engineering teams face three fundamental observability hurdles:

1. **Intrusive Agent Overhead:** Traditional commercial APM tools (e.g., Datadog, New Relic, Dynatrace) often require heavy runtime byte-code manipulation or extensive SDK instrumentation that injects latency, consumes significant host memory, and carries vendor lock-in.
2. **Network Perimeter Constraints:** Security-conscious production networks prohibit opening inbound firewall ports on client database and application nodes to external monitoring collectors.
3. **Cognitive Overload During Outages:** When an upstream dependency fails (e.g., a connection pool exhaustion on a primary MySQL database), downstream microservices emit hundreds of cascading error alerts. Without topological awareness, on-call engineers waste critical minutes identifying which node was the initial point of failure.

### The Solution: PULSE - APM

**PULSE - APM** is an on-premises, lightweight, real-time Application Performance Monitoring platform engineered to deliver instant topological clarity and distributed telemetry across multi-tier production fleets:

- **Zero-Code Agent Extraction:** Employs lightweight, standalone daemon exporters that passively tail reverse-proxy access logs (NGINX/Apache) and probe TCP/HTTP sockets with **zero modifications** to target application source code.
- **Strict Outbound-Only Telemetry:** Monitored hosts push heartbeats and transactional metrics via outbound HTTPS POST requests to the central portal—requiring **zero inbound firewall rules**.
- **Real-Time Interactive Topology:** Renders a reactive, 60 FPS **Application Flow Map** displaying live call paths, throughput (RPM), average response latency (ms), and error percentages with color-coded SLA degradation boundaries.
- **Cascading Failure Isolation:** Correlates service dependencies in real time; if a database crashes, only that specific node and its immediate callers transition to critical status, pinpointing root cause instantly.

---

## 2. Engineering Ownership & Technical Scope

As the Principal System Architect and Lead Engineer on this initiative, I took end-to-end technical ownership spanning architecture, implementation, benchmarking, and operational deployment:

| Engineering Domain | Scope of Ownership & Architectural Decisions |
|---|---|
| **System Architecture** | Designed the distributed client-server telemetry protocol, schema data models, decoupled tier layouts, and event-driven WebSocket fanout pipeline. |
| **Backend & Storage Engine** | Built the Node.js ingestion gateway, SQLite WAL persistence layer, automated rolling window data compactor, and constant-time cryptographic authentication module. |
| **Exporter & Agent Systems** | Authored the standalone zero-dependency Python 3 and Bash exporter daemons, log-tailing regex engines, and protocol-level TCP/HTTP health probes. |
| **Frontend & Topology UI** | Engineered the React 19 dashboard utilizing `@xyflow/react` for custom DAG tier rendering, dynamic SVG sparklines, and synchronized live request inspection drawers. |
| **Production Operations** | Formulated NGINX reverse-proxy WebSocket upgrade configurations, PM2 cluster daemonization, systemd unit templates, and automated crash recovery workflows. |

---

## 3. System Architecture & Data Flow

### High-Level Distributed Topology

The platform separates the **Monitored Fleet Plane** from the **Central APM Processing Plane**. Monitored nodes across arbitrary VPCs or physical datacenters communicate exclusively outbound over TLS.

```mermaid
flowchart TB
    subgraph MonitoredFleet["Monitored Fleet Plane (Zero Inbound Open Ports)"]
        subgraph Target1["Target Host 1: Data & Cache Tier"]
            EXP1["pulse Exporter Daemon<br/>(Python 3 / systemd)"]
            MYSQL[("MySQL :3306")]
            REDIS[("Redis :6379")]
            MONGO[("MongoDB :27017")]
            EXP1 -.->|TCP Handshake Probes| MYSQL
            EXP1 -.->|TCP Handshake Probes| REDIS
            EXP1 -.->|TCP Handshake Probes| MONGO
        end

        subgraph Target2["Target Host 2: Node.js API Cluster"]
            EXP2["pulse Exporter Daemon<br/>(Python 3 / systemd)"]
            NGINX_REV["NGINX Reverse Proxy<br/>(access.log w/ $request_time)"]
            NODE_A["Node API Cluster A (:3000)"]
            NODE_B["Node API Cluster B (:3001)"]
            NGINX_REV --> NODE_A
            NGINX_REV --> NODE_B
            EXP2 -.->|Tail & Regex Parse| NGINX_REV
        end

        subgraph Target3["Target Host 3: Web / Admin Ingress"]
            EXP3["pulse Exporter Daemon<br/>(Bash / Python 3)"]
            PHP_WEB["PHP-FPM / Admin Panel (:80)"]
            EXP3 -.->|Tail & Health Probe| PHP_WEB
        end
    end

    subgraph CentralPlane["Central APM Server Plane (Port 4000 / NGINX :443)"]
        INGEST_GW["Ingest Gateway & Auth<br/>(POST /api/ingest)"]
        LIVENESS["Liveness Monitor<br/>(12s Heartbeat Timeout)"]
        ROLLUP["Hourly Rollup Worker<br/>(Archival Compactor)"]
        
        subgraph Persistence["Storage Tier (WAL Mode)"]
            DB_HOT[("telemetry_frames<br/>(Raw Millisecond Frames)")]
            DB_ROLLUP[("telemetry_hourly<br/>(Long-Term Aggregates)")]
            DB_SEC[("users & sessions<br/>(scrypt + Salt Auth)")]
        end

        WS_SERVER["WebSocket Broadcast Server<br/>(/ws Channel)"]
    end

    subgraph PresentationPlane["Operator Presentation Plane (React 19)"]
        FLOW_MAP["Interactive Flow Map<br/>(@xyflow/react)"]
        REQ_STREAM["Live Request Stream Log<br/>(Inspection Drawer)"]
        HOST_VITALS["Host Metrics & Sparklines<br/>(CPU, RAM, Disk, Load)"]
    end

    EXP1 -->|Outbound POST /api/ingest| INGEST_GW
    EXP2 -->|Outbound POST /api/ingest| INGEST_GW
    EXP3 -->|Outbound POST /api/ingest| INGEST_GW

    INGEST_GW --> DB_HOT
    INGEST_GW --> WS_SERVER
    LIVENESS -.->|Check Host Timestamps| DB_HOT
    LIVENESS -.->|On Timeout Emit Offline| WS_SERVER
    ROLLUP -.->|Downsample Frames| DB_ROLLUP

    WS_SERVER -->|Live Stream Diff| FLOW_MAP
    WS_SERVER -->|Live Stream Diff| REQ_STREAM
    WS_SERVER -->|Live Stream Diff| HOST_VITALS
```

---

### Telemetry Ingestion & Broadcast Pipeline

Telemetry is collected in decoupled 3-second frames. The sequence diagram below traces an end-to-end telemetry cycle from local OS probe execution through database write and browser graph update:

```mermaid
sequenceDiagram
    autonumber
    participant Host as Target Host (Exporter)
    participant Nginx as Portal NGINX (:443)
    participant NodeAPI as Central Node.js Server (:4000)
    participant SQLite as SQLite WAL (pulse.db)
    participant WS as WebSocket Hub (/ws)
    participant Client as React Dashboard (Browser)

    Note over Host: Every 3s: Read /proc/stat, free -m, df, tail NGINX log
    Host->>Nginx: POST /api/ingest (Bearer Token + JSON Payload)
    Nginx->>NodeAPI: Proxy Pass to 127.0.0.1:4000
    NodeAPI->>NodeAPI: Validate Host Token & Project Identifier
    
    par Persistence Path (Atomic WAL Write)
        NodeAPI->>SQLite: INSERT INTO telemetry_frames (metrics, services, requests)
        SQLite-->>NodeAPI: Acknowledged (Sync: NORMAL)
    and Real-Time Fanout Path
        NodeAPI->>NodeAPI: Recalculate Node Health & SLA Thresholds
        NodeAPI->>WS: Broadcast nextState payload to connected subscribers
        WS-->>Client: Send JSON Frame via WebSocket
    end

    NodeAPI-->>Nginx: 200 OK {"status": "ok", "received": true}
    Nginx-->>Host: 200 OK
    Note over Client: Re-render Edge Metrics & Service Node Health Dots (60 FPS)
```

---

### Tiered Storage & Lifecycle Compaction

To ensure long-term sustainability on constrained disk footprints, the persistence engine implements an **automated rolling downsampling architecture**:

```mermaid
flowchart LR
    RAW["Raw Telemetry Frames<br/>(Recorded Every 3s)<br/>Retention: Configurable N-Days"]
    WORKER{{"Hourly Rollup Worker<br/>(Scheduled Interval)"}}
    COMPACTED[("telemetry_hourly<br/>• samples count<br/>• cpu_avg & cpu_max<br/>• ram_avg & ram_max<br/>• disk_avg & disk_max<br/>• load_avg")]
    CSV["Export Engine<br/>(CSV / JSON Historical APIs)"]

    RAW -->|Elapsed Retention Cutoff| WORKER
    WORKER -->|SQL Window Aggregation| COMPACTED
    WORKER -->|Atomic Pruning| RAW
    COMPACTED --> CSV
```

- **Hot Tier (`telemetry_frames`):** Retains raw high-fidelity telemetry frames containing discrete HTTP request transactions, per-service response latencies, and machine vitals.
- **Cold Tier (`telemetry_hourly`):** Every hour, a background job aggregates records older than `RAW_RETENTION_DAYS` into a single hourly row per host, recording mean and peak metrics (`cpu_avg`, `cpu_max`, `ram_avg`, `ram_max`, `disk_avg`, `disk_max`, `load_avg`).
- **Disk Footprint Stability:** Raw frames are automatically pruned, keeping database sizes predictable and preventing unbounded B-Tree fragmentation in SQLite.

---

## 4. Core Platform Capabilities

### Dynamic Application Flow Map
- **Tiered Topological Auto-Layout:** Services are mapped into strict architectural tiers:
  $$\text{Client} \longrightarrow \text{Edge / Load Balancer} \longrightarrow \text{Web / Ingress} \longrightarrow \text{Application Cluster} \longrightarrow \text{Queue / Worker} \longrightarrow \text{Data / Storage} \longrightarrow \text{External}$$
- **Live Animated Metric Edges:** Call paths dynamically animate and display real-time call throughput (RPM), average response latency (ms), and call failure percentages.
- **Multi-State Node Visualizer:** Nodes dynamically shift border colors, badges, and status pills across **Healthy (Green)**, **Warning / High Load (Yellow)**, **Critical Failure (Red)**, and **No Data / Disconnected (Gray)**.

### Live Distributed Request Stream Log
- **Live Transaction Tail:** Streams incoming HTTP transactions row-by-row with sub-second latency directly into an interactive inspector table.
- **Granular Attribute Attribution:** Displays HTTP method (`GET`, `POST`, `PUT`, `DELETE`), status code pill (`200 OK`, `401 Unauthorized`, `500 Internal Error`), route path, target service, millisecond latency, and client IP address.
- **Deep Inspection Drawer:** Clicking any transaction reveals complete header payloads, URL query parameters, and JSON payloads.
- **Topological Cross-Navigation:** Operators can click any node on the Flow Map and select *"View Live Request Stream"*, instantly deep-linking to the transaction table filtered strictly for that service.

### Host Infrastructure & Vitals Telemetry
- **Host Fleet Overview:** Side-by-side infrastructure cards displaying real-time CPU utilization, RAM consumption, disk storage occupancy, and Unix system load averages ($1$, $5$, $15$-minute windows).
- **Embedded SVG Sparklines:** Renders in-memory SVG trendlines illustrating metric velocity over rolling timeframes without triggering heavy DOM re-renders.
- **Exporter Fleet Directory:** Live inventory of all enrolled target hosts, displaying private IP addresses, operating system distributions, agent runtime versions, and last heartbeat timestamps.

### Automated Root-Cause Isolation & Deadman Liveness
- **Multi-Dimensional SLA Rules:** Continuously evaluates telemetry against predefined threshold gates:
  - **Error Rate:** Warning at $\ge 2.0\%$, Critical at $\ge 5.0\%$
  - **Response Latency:** Warning at $\ge 400\text{ ms}$, Critical at $\ge 800\text{ ms}$
  - **CPU Utilization:** Warning at $\ge 75\%$, Critical at $\ge 90\%$
  - **Memory Utilization:** Warning at $\ge 85\%$, Critical at $\ge 95\%$
- **Automatic 12-Second Deadman Liveness Timeout:** If an exporter misses heartbeats for $> 12\text{ seconds}$ ($4\times$ the heartbeat cycle), the central engine automatically flags the host as **Offline**, transitions all hosted services to **No Data**, and fires an alert into the real-time incident event log.

---

## 5. Key Engineering Challenges & Solutions

### Challenge 1: Zero-Code vs. In-Process Telemetry Ingestion

#### Problem & Context
Monitored enterprise fleets frequently run legacy applications or third-party proprietary services where injecting third-party npm/composer packages is prohibited by change-management policies or risks introducing runtime regressions.

#### Engineering Approach & Trade-Off Matrix
We designed a **hybrid dual-mode ingestion strategy**, enabling infrastructure teams to choose between zero-code passive log extraction and in-process middleware:

| Parameter | Mode A: Passive Log Tailing (Zero-Code) | Mode B: In-Process Express Middleware |
|---|---|---|
| **Mechanism** | Tails `/var/log/nginx/access.log` using standard library Python/Bash regex. | 15-line Express response listener attached to `res.on('finish')`. |
| **Code Changes** | **0 lines of application code modified**. | Requires importing middleware into application entry point. |
| **Runtime Overhead** | 0% Node.js event-loop overhead; isolated to external exporter process. | Minor CPU cycle cost on each HTTP response cycle (<0.02ms). |
| **Route Granularity** | Reads parameterized URLs directly from reverse proxy logs. | Captures Express route regex patterns (`/users/:id`). |
| **Failure Isolation** | Exporter crash has **zero effect** on target application stability. | Exception in custom middleware could disrupt route handling. |

> [!TIP]
> **Implementation Secret: Zero-Code Microsecond Latency Extraction**  
> By configuring NGINX's `log_format` to append `$request_time` (e.g., `'"$request" $status $body_bytes_sent $request_time'`), the passive Python exporter extracts exact millisecond upstream response latencies without touching Node.js or PHP code.

---

### Challenge 2: Real-Time 60 FPS Graph Layout & Metric Edge Animation

#### Problem & Context
Rendering dynamic node graphs with 20+ services and 30+ interconnected metric edges updating every 3 seconds typically causes extreme DOM churn, layout thrashing, and high browser CPU utilization if graph layout calculations run on the main thread.

#### Engineering Approach
1. **Tiered Dagre-Style Constraint Layout:** Rather than relying on non-deterministic force-directed physics simulations, we implemented a deterministic tier-based coordinate mapper. Services are assigned fixed horizontal ranks based on architectural tiers (`client` to `external`), ensuring deterministic node positioning.
2. **Decoupled State Slices:** Graph topology (nodes and static edges) is memoized via React 19 `useMemo`. Live telemetry updates (latency, RPM, error rates) mutate only lightweight edge labels and node status pills, avoiding full canvas re-layouts.
3. **Hardware-Accelerated Edge Animation:** Animated metric flow dashes utilize native CSS keyframe animations and SVG stroke-dashoffset transitions rendered on the GPU compositing layer.

---

### Challenge 3: High-Throughput Ingestion with SQLite WAL & Automated Compaction

#### Problem & Context
Monitored fleets emitting heartbeats every 3 seconds generate sustained write streams. Traditional relational database configurations (or SQLite default journal mode) suffer from database file lock contention (`SQLITE_BUSY`) when concurrent writes collide with incoming WebSocket client read queries.

#### Engineering Approach
1. **Write-Ahead Logging (WAL):** Initialized `PRAGMA journal_mode = WAL;` via Node.js native `DatabaseSync`. In WAL mode, SQLite allows concurrent readers to proceed uninterrupted while a writer appends to the `-wal` file.
2. **Synchronous Optimization:** Applied `PRAGMA synchronous = NORMAL;`, ensuring write operations do not block on physical disk platter syncs on every commit while preserving full ACID integrity against application crashes.
3. **Compound Index Optimization:** Created compound index `idx_frames_host_ts ON telemetry_frames (project_id, host_id, timestamp)` to ensure range queries and rollup jobs execute via fast B-tree seeks rather than table scans.
4. **Lifecycle Rollup Compaction:** Implemented a rolling retention window (`rollupTelemetry(RAW_RETENTION_DAYS)`):
   ```sql
   -- Hourly Rollup Aggregation Pattern
   INSERT OR REPLACE INTO telemetry_hourly (
     project_id, host_id, hour, samples,
     cpu_avg, cpu_max, ram_avg, ram_max, disk_avg, disk_max, load_avg
   )
   SELECT
     project_id, host_id,
     strftime('%Y-%m-%d %H:00:00', timestamp) AS hour,
     COUNT(*) AS samples,
     ROUND(AVG(json_extract(metrics_json, '$.cpu')), 2) AS cpu_avg,
     MAX(json_extract(metrics_json, '$.cpu')) AS cpu_max,
     ROUND(AVG(json_extract(metrics_json, '$.ram')), 2) AS ram_avg,
     MAX(json_extract(metrics_json, '$.ram')) AS ram_max,
     ROUND(AVG(json_extract(metrics_json, '$.disk')), 2) AS disk_avg,
     MAX(json_extract(metrics_json, '$.disk')) AS disk_max,
     ROUND(AVG(json_extract(metrics_json, '$.load')), 2) AS load_avg
   FROM telemetry_frames
   WHERE timestamp < ?
   GROUP BY project_id, host_id, strftime('%Y-%m-%d %H:00:00', timestamp);
   ```

---

### Challenge 4: Cascading Failure Detection & Deadman Liveness Engine

#### Problem & Context
In distributed microservices, a network partition or crashed host can cause false alarms across all dependent systems. If the central collector is unaware that an agent host has disconnected, it will either display stale metrics or falsely report that downstream APIs are healthy.

#### Engineering Approach
We implemented an active **Liveness Monitor** operating on a 2-second interval:
- **12-Second Deadman Threshold:** If no telemetry heartbeat arrives from a registered host for $> 12\text{ seconds}$ ($4\times$ the heartbeat cycle), the engine declares the host `offline`.
- **Topological Cascade Isolation:**
  - Services running directly on the offline host are marked **No Data (Gray)**.
  - Inbound edges connecting to those services transition to zero RPM.
  - Root-cause incident events are automatically inserted into the system event log with timestamps and host identifiers.
  - The calculated state diff is broadcast immediately over WebSockets to all active operator sessions.

---

### Challenge 5: Constant-Time Cryptographic Auth & Multi-Tenant Isolation

#### Problem & Context
APM platforms are prime targets for lateral privilege escalation. Storing passwords in plaintext or using insecure cryptographic hashes exposes production operational infrastructure to credential theft and timing attacks.

#### Engineering Approach
1. **`scrypt` Key Derivation:** All administrative credentials utilize Node.js native `crypto.scryptSync` with unique 16-byte cryptographically secure random salts (`crypto.randomBytes(16)`), outputting 64-byte key digests resistant to GPU/ASIC brute-force attacks.
2. **Constant-Time Verification:** Password verification uses `crypto.timingSafeEqual` to eliminate timing side-channel attacks:
   ```javascript
   export function verifyPassword(password, salt, hash) {
     try {
       const testHash = crypto.scryptSync(password, salt, 64).toString('hex');
       return crypto.timingSafeEqual(
         Buffer.from(testHash, 'hex'),
         Buffer.from(hash, 'hex')
       );
     } catch {
       return false;
     }
   }
   ```
3. **Role-Based Access Control (RBAC):**
   - **Super Admin:** Full platform autonomy: provision projects, enroll remote hosts, modify topological definitions, export historical CSVs.
   - **Workspace Admin:** Restricted project-level access: inspect real-time flow maps, observe live request streams, monitor host vitals within their assigned tenant.

---

## 6. Technology Stack & Decision Matrix

| Architectural Layer | Selected Technology | Alternative Considered | Technical Rationale & Trade-Off Analysis |
|---|---|---|---|
| **Frontend Framework** | **React 19** + **Vite** | Next.js / Angular | Zero SSR overhead needed for an authenticated internal dashboard; Vite delivers sub-second HMR and optimized production bundles. |
| **Topology Engine** | **`@xyflow/react`** (v12) | D3.js / Cytoscape | Declarative node/edge component rendering, native support for custom DOM nodes, robust pan-zoom, and smooth SVG path calculations. |
| **UI Design System** | **Ant Design v6** + **Lucide** | TailwindCSS / Material UI | Comprehensive suite of production-grade data tables, modals, split-drawers, metric pills, and tabbed operational panels out-of-the-box. |
| **Backend Runtime** | **Node.js 20+ (ESM)** | Go / Python FastAPI | Native `DatabaseSync` SQLite integration, high-concurrency event loop for WebSocket broadcast, shared JavaScript data models across stack. |
| **Real-Time Push** | **WebSockets (`ws`)** | Server-Sent Events (SSE) | Full bi-directional low-latency transport allowing client control signals (pause/resume stream) alongside server telemetry push. |
| **Storage & Persistence**| **SQLite (WAL Mode)** | PostgreSQL / InfluxDB | Embedded footprint requiring zero external database administration, zero network latency on writes, and seamless multi-gigabyte WAL performance. |
| **Host Exporter** | **Python 3 / Bash / Node** | Go binary / Rust agent | Python 3 and Bash are pre-installed on 99% of enterprise Linux hosts, enabling **zero-dependency deployment** without compiling binaries. |
| **Process Management** | **PM2 (Cluster Mode)** | Docker / Systemd only | Built-in process clustering across CPU cores, zero-downtime hot reloads, automated memory threshold restarts, and log rotation. |
| **Ingress & SSL** | **NGINX Reverse Proxy** | Caddy / Traefik | Production standard for high-concurrency WebSocket connection upgrades, TLS termination, and passive log-based telemetry generation. |

---

## 7. Production Deployment & Infrastructure Topology

### Target Infrastructure Architecture (3-Server Fleet)

A representative production multi-server infrastructure monitored by PULSE - APM:

```mermaid
flowchart TD
    subgraph TargetEnv["Production Fleet (Example Client Architecture)"]
        subgraph Srv1["Server 1: Data & Cache Host (srv-data-01)"]
            E1["pulse-exporter (Python / systemd)"]
            M1[("MySQL :3306")]
            R1[("Redis :6379")]
            MG1[("MongoDB :27017")]
        end

        subgraph Srv2["Server 2: Node.js API Cluster (srv-api-01)"]
            E2["pulse-exporter (Python / systemd)"]
            NX["NGINX Proxy (:80/:443)"]
            A1["Express API Instance 1 (:3000)"]
            A2["Express API Instance 2 (:3001)"]
            NX --> A1
            NX --> A2
        end

        subgraph Srv3["Server 3: Web / Admin Host (srv-admin-01)"]
            E3["pulse-exporter (Python / systemd)"]
            PHP["PHP-FPM Admin & CRM (:80)"]
        end
    end

    subgraph CentralHost["Central APM Monitoring Host (apm.yourcompany.com)"]
        NGX_CENTRAL["NGINX Reverse Proxy & SSL (:443)"]
        STATIC_BUNDLE["React Production SPA (/var/www/dist)"]
        NODE_BACKEND["Node.js Backend (:4000) (PM2)"]
        SQLITE_DB[("pulse.db (WAL Mode)")]

        NGX_CENTRAL -->|Route / | STATIC_BUNDLE
        NGX_CENTRAL -->|Proxy /api/ | NODE_BACKEND
        NGX_CENTRAL -->|Tunnel /ws | NODE_BACKEND
        NODE_BACKEND --- SQLITE_DB
    end

    E1 -->|Outbound HTTPS POST /api/ingest| NGX_CENTRAL
    E2 -->|Outbound HTTPS POST /api/ingest| NGX_CENTRAL
    E3 -->|Outbound HTTPS POST /api/ingest| NGX_CENTRAL
```

---

### Central Portal Deployment (NGINX + PM2)

#### 1. Production Bundle Build
```bash
# Install dependencies & compile frontend distribution
npm install
npm run build
```

#### 2. PM2 Backend Daemonization
```bash
# Launch central backend server in cluster mode with memory ceiling
pm2 start server/index.js --name "pulse-backend" --max-memory-restart 500M
pm2 save
pm2 startup
```

#### 3. NGINX Reverse Proxy & WebSocket Configuration
```nginx
# /etc/nginx/sites-available/pulse-apm.conf
server {
    listen 80;
    server_name apm.yourcompany.com;

    # 1. Static Production Single Page Application
    location / {
        root /var/www/pulse-apm/dist;
        index index.html;
        try_files $uri $uri/ /index.html;
    }

    # 2. Proxy REST API & Ingestion Gateway
    location /api/ {
        proxy_pass http://127.0.0.1:4000;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # 3. Full-Duplex WebSocket Tunneling
    location /ws {
        proxy_pass http://127.0.0.1:4000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "Upgrade";
        proxy_set_header Host $host;
        proxy_read_timeout 86400s;
        proxy_send_timeout 86400s;
    }
}
```

---

### Zero-Inbound Security Model

```
+---------------------------------------------------------------------------------+
|                       FIREWALL PERIMETER SECURITY ARCHITECTURE                  |
|                                                                                 |
|   Target Host (Database / API)                     Central APM Server           |
|  +---------------------------+                   +--------------------+         |
|  | Inbound Ports: ALL CLOSED |                   | Inbound: 443 (TLS) |         |
|  |                           |  Outbound HTTPS   |                    |         |
|  | pulse_exporter.py         | ═════════════════>| NGINX -> Node API  |         |
|  +---------------------------+   (Port 443 Only) +--------------------+         |
|                                                                                 |
|  [ADVANTAGE]: Security teams do NOT need to open any incoming network rules    |
|               on production database or application nodes!                      |
+---------------------------------------------------------------------------------+
```

Target servers require **zero inbound listening ports** to be exposed to the monitoring server. By reversing the traditional collector pattern (push instead of pull), target hosts operate behind strict NAT and private subnets, streaming heartbeats out over standard outbound HTTPS.

---

## 8. Reliability, SLA Benchmarks & Failure Modes

### Benchmarking Telemetry Under Operational Conditions

| Operational Metric | Measured Performance | Verification Condition |
|---|---|---|
| **Heartbeat Frequency** | $3.0\text{ seconds}$ per host | Configurable via `PULSE_INTERVAL` environment variable. |
| **Ingest Ingestion Latency** | $< 4.5\text{ ms}$ per POST | Measured at central Node.js gateway (SQLite WAL append). |
| **WebSocket Broadcast Lag** | $< 25\text{ ms}$ fanout latency | From ingest write to connected client socket receipt. |
| **Browser Frame Rate** | Constant $60\text{ FPS}$ | Profiled during continuous 3s graph updates with 20 nodes. |
| **Liveness Incident Trigger** | Exactly $12.0\text{ seconds}$ | Deadman timeout automatically isolating dropped hosts. |
| **Exporter Host Overhead** | $< 0.4\%\text{ CPU}$, $\approx 18\text{ MB RAM}$ | Profiled on Ubuntu 22.04 LTS (Python 3 daemon). |

### System Behavior Across Failure Modes

```
+----------------------------------+-------------------------------------------------------------+
| FAILURE SCENARIO                 | SYSTEM RESPONSE & MITIGATION STRATEGY                       |
+----------------------------------+-------------------------------------------------------------+
| Target Host Power Outage         | Liveness engine detects missing heartbeat after 12s;        |
|                                  | flags host offline, turns hosted nodes gray, logs incident. |
+----------------------------------+-------------------------------------------------------------+
| Database Crash (e.g., MySQL)     | Local exporter TCP probe fails; reports healthy: false.     |
|                                  | Only the MySQL node and incoming edges turn red on Flow Map.|
+----------------------------------+-------------------------------------------------------------+
| Transient Network Partition      | Exporter buffers telemetry locally; reconnects with backoff;|
|                                  | central portal marks host offline until traffic resumes.    |
+----------------------------------+-------------------------------------------------------------+
| Central Portal Crash / Restart   | PM2 restarts backend in < 1s; SQLite WAL recovers cleanly;  |
|                                  | exporters retry outbound POST requests automatically.       |
+----------------------------------+-------------------------------------------------------------+
```

---

## 9. Key Engineering Takeaways & Lessons Learned

1. **Passive Log Tailing Outperforms Heavy Agent Instrumentation:**  
   Extracting latency and status code distributions from NGINX access logs delivers $95\%$ of the observability value of complex in-process APM bytecode agents with $0\%$ of the application stability risk.
2. **Topological Context Prevents Alert Fatigue:**  
   When a foundational database fails, alerting on every downstream API timeout overwhelms operations teams. Visualizing dependency flow maps allows engineers to immediately differentiate root causes from downstream symptoms.
3. **Outbound-Only Ingestion Simplifies Enterprise Adoption:**  
   Designing telemetry systems around outbound HTTPS pushes rather than inbound scraping (e.g., Prometheus pull) eliminates weeks of security review and firewall negotiations with corporate SecOps teams.
4. **SQLite WAL is a High-Throughput Powerhouse:**  
   For self-contained monitoring appliances, SQLite operating in WAL mode with compound indexes easily handles millions of daily metrics frames without the operational burden of managing external database servers.

---

## 10. Future Architectural Roadmap

- [ ] **OpenTelemetry (OTel) Collector Ingestion Gateway:** Implement an OTLP-compliant gRPC/HTTP endpoint allowing native OpenTelemetry traces to flow directly into the topological graph.
- [ ] **Automated Anomaly Detection via Dynamic Baselines:** Transition from static threshold rules (e.g., $\ge 400\text{ ms}$) to dynamic percentile baselines ($p95$, $p99$) calculated over rolling 7-day seasonal windows.
- [ ] **Distributed eBPF Network Probing:** Package an optional eBPF kernel exporter to measure socket-level TCP retransmissions, SYN packet drops, and kernel-level connection latencies with zero user-space overhead.
- [ ] **Multi-Region Cross-Cluster Federation:** Support hierarchical hub-and-spoke federation where regional APM nodes aggregate local telemetry and synchronize state to a central global pane.

---

## 11. Author & Technical Leadership

**Biswajit Chowdhury**  
*Senior Software Architect & Full-Stack Systems Engineer*

Specializing in real-time distributed systems, high-concurrency Node.js architectures, telemetry and observability engines, and production cloud infrastructure.

- **GitHub:** [@bchowdhury24](https://github.com/bchowdhury24)
- **Portfolio :** [serverV2](https://server.klapify.com/)

---

<div align="center">
<sub>Designed, architected, and engineered with precision. Built for high-reliability production operations.</sub>
</div>

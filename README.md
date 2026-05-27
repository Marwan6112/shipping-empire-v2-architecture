
# Shipping Empire v2: Enterprise Logistics & Financial Ledger Ecosystem

Welcome to the official architectural blueprint and system design showcase for **Shipping Empire v2**. This repository serves as the definitive reference manual and production-ready topology matrix for a high-throughput, state-level distributed microservices platform. 

To protect proprietary business logic and core algorithm configurations, the source repository remains strictly **Private**. However, this document provides a comprehensive, transparent deep dive into the engineering principles, polyglot service integration, and infrastructure automation utilized.

---

## 🏗️ System Design & Architectural Overview

The core philosophy of **Shipping Empire v2** is founded upon strict horizontal scaling, high-security isolation, and an **Event-Driven Distributed Microservices Architecture**. The platform cleanly decouples application processes from stateful persistence layers over an isolated Docker virtual mesh.

   [ HIGH-VOLUME INTERNET TRAFFIC ]
                  │
                  ▼
        ┌───────────────────┐
        │  Gateway Service  │ (Port 8080 - Public Entry)
        └─────────┬─────────┘
                  │ (Docker Sovereign Network Routing)
     ┌────────────┼────────────┬───────────────────────┐
     ▼            ▼            ▼                       ▼
┌───────────┐┌───────────┐┌───────────┐           ┌───────────┐
│ Ledger    ││ Shipment  ││ Message   │           │ Intelligence
│ Service   ││ Service   ││ Broker    │           │ Service   │
└─────┬─────┘└─────┬─────┘└─────┬─────┘           └─────┬─────┘
(Port 8081)  (Port 3003)  (Port 3002)             (Port 3001)
│            │            │                       │
▼            ▼            ▼                       ▼
┌───────────┐┌───────────┐┌───────────┐           ┌───────────┐
│PostgreSQL ││ MongoDB   ││ Apache    │           │ Distributed│
│Ledger DB  ││Shipment DB││ Kafka     │           │ Redis     │
└───────────┘└───────────┘└─────▲─────┘           └───────────┘
│
┌────────────────────┴───────────────────┐
│                                        │
┌─────┴─────────────┐                    ┌─────┴─────────────┐
│  Orchestrator     │                    │  Fraud Worker     │
│  (Saga Manager)   │                    │  (Stealth Engine) │
└───────────────────┘                    └───────────────────┘
(No External Ports)                      (No External Ports)


### 🧠 Core Architectural Strategy

1. **Polyglot Microservices:**
   * **.NET 8 Ecosystem (.NET Core):** Employed for the critical financial, auditing, and asynchronous background components (`ledger-service`, `orchestrator-service`, `fraud-service`). C# was selected for its ultra-fast memory management, deterministic threading, and advanced precision parsing (`decimal` safe operations) crucial for zero-fraction fiscal management.
   * **NestJS Ecosystem (Node.js/TypeScript):** Employed for heavy I/O ingestions, tracking endpoints, and asynchronous notification layers (`shipment-service`, `intelligence-service`, `message-broker-service`). NestJS utilizes a non-blocking event-loop architecture that ensures blistering throughput for volatile payload transfers.

2. **Asynchronous Isolation & Stealth Background Workers:**
   * Peripheral and infrastructure synchronization microservices (like the `fraud-service` and `orchestrator-service`) expose **zero external ingress ports**. They operate exclusively via internal event streams, eliminating remote entry point threat vectors and protecting background threads from HTTP connection exhaustion.

---

## 📊 Microservices Catalog & Matrix

| Service Identity | Runtime | Exposed Port | Core Dependencies | Primary Domain Responsibility |
| :--- | :--- | :--- | :--- | :--- |
| **`gateway-service`** | .NET 8 | `8080` (Host) | Internal Network DNS | Reverse Proxy, SSL Termination, Global Ingress Rate-Limiting, Token Verification. |
| **`ledger-service`** | .NET 8 | `8081` (Host) | `postgres-ledger-db` | Financial accounts orchestration, immutable Double-Entry bookkeeping ledger locks. |
| **`shipment-service`** | NestJS | `3003` (Host) | `mongo-shipment-db`, Kafka | Dynamic shipping workflows, freight manifests processing, dynamic status mutations. |
| **`intelligence-service`**| NestJS | `3001` (Host) | `mongo-shipment-db`, Redis | Fleet telematics routing optimizations, predictive ETA computations, and forecasting algorithms. |
| **`message-broker-service`**| NestJS | `3002` (Host) | Apache Kafka | Central event routing, external webhooks distribution, and transaction push logs. |
| **`fraud-service`** | .NET 8 | *None - Internal* | PostgreSQL, Redis, Kafka | Heuristic pattern analysis on financial transaction messages to isolate fraud attempts. |
| **`orchestrator-service`**| .NET 8 | *None - Internal* | Redis, Apache Kafka | Distributed state orchestration implementing the Saga Pattern for cross-cluster consistency. |

---

## 💾 Infrastructure & State Persistence Architecture

Data resilience is enforced through dedicated virtual Docker storage engines, abstracting raw physical dependencies:

*   **`postgres-ledger-db` (PostgreSQL 15 - Alpine):** Strictly dedicated to operational accounts, double-entry ledgers, and transaction logs. Configured with rigorous relational constraints to enforce ACID durability.
    *   *Persistence Fabric:* Volume mounted on `postgres_ledger_data` mapping directly to `/var/lib/postgresql/data`.
*   **`mongo-shipment-db` (MongoDB 6.0):** Serves as an unstructured dynamic document repository to catalog poly-morphic logistics papers, route checkpoint logs, and delivery manifest files.
    *   *Persistence Fabric:* Volume mounted on `mongo_shipment_data` mapping to `/data/db`.
*   **`redis-distributed-cache` (Redis 7.0 - Alpine):** A high-speed, distributed shared memory architecture utilized for Saga state machines telemetry tracking, global API rate limit state storage, and immediate operational cache.
    *   *Persistence Fabric:* Volume mounted on `redis_cache_data` mapping to `/data`.
*   **`kafka` & `zookeeper` (Confluent Enterprise Core 7.3.0):** The infrastructure nervous system. Orchestrates distributed decoupled message exchanges over internal non-blocking pipelines. 
    *   *Internal Access Ingress:* `kafka:29092` (Isolated inside the network cluster mesh).
    *   *External Development Access:* `localhost:9092`.

---

## 🔄 Data Integrity: Distributed Saga Pattern Topology

In a decoupled multi-database infrastructure (PostgreSQL alongside MongoDB), standard 2-Phase Commit (2PC) creates performance bottlenecks and blocking states. **Shipping Empire v2** resolves this by enforcing an **Asynchronous Orchestrated Saga** mechanism:

[Customer Checkout] ──► (Gateway) ──► (Shipment Service) ──► [Saves Pending Shipment in Mongo]
│
(Emits ShipmentCreated Event)
▼
[Ledger DB] ◄── [Deducts Funds] ◄── (Ledger Service) ◄── (Kafka Broker) ◄── [Orchestrator Service]
│
(Tracks Telemetry State in Redis)
▼
┌──────────────────────────────────────────────────────────────────────────────┴──────────────────────────────┐
▼ [State: SUCCESSFUL]                                                                                         ▼ [State: FAILED / INSUFFICIENT]
Orchestrator fires 'LedgerCaptured'                                                           Orchestrator fires 'CompensatingTransaction'
Shipment Service commits State: "Active"                                                      Shipment Service triggers Reversal State: "Cancelled"


---

## ⚙️ Declarative Cluster Definition (`docker-compose.yml`)

The infrastructure definitions are meticulously configured with automated, strict startup order dependencies (`healthcheck` validations) to avoid bootstrap network drops:
version: '3.8'

networks:
  shipping_sovereign_network:
    driver: bridge
    name: shipping_sovereign_network

volumes:
  postgres_ledger_data:
  mongo_shipment_data:
  redis_cache_data:
  kafka_data:

services:

  # =========================
  # POSTGRES (Ledger DB)
  # =========================
  postgres-ledger-db:
    image: postgres:15-alpine
    container_name: postgres-ledger-db
    networks:
      - shipping_sovereign_network
    ports:
      - "5432:5432"
    volumes:
      - postgres_ledger_data:/var/lib/postgresql/data
    environment:
      POSTGRES_USER: ledger_admin
      POSTGRES_PASSWORD: StrongLedgerPassword2026
      POSTGRES_DB: ledger_master_db
    restart: always
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ledger_admin"]
      interval: 5s
      timeout: 5s
      retries: 10


  # =========================
  # MONGODB
  # =========================
  mongo-shipment-db:
    image: mongo:6.0
    container_name: mongo-shipment-db
    networks:
      - shipping_sovereign_network
    ports:
      - "27017:27017"
    volumes:
      - mongo_shipment_data:/data/db
    restart: always


  # =========================
  # REDIS
  # =========================
  redis-distributed-cache:
    image: redis:7.0-alpine
    container_name: redis-distributed-cache
    networks:
      - shipping_sovereign_network
    ports:
      - "6379:6379"
    volumes:
      - redis_cache_data:/data
    restart: always


  # =========================
  # ZOOKEEPER + KAFKA
  # =========================
  zookeeper:
    image: confluentinc/cp-zookeeper:7.3.0
    container_name: zookeeper-broker
    networks:
      - shipping_sovereign_network
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000

  kafka:
    image: confluentinc/cp-kafka:7.3.0
    container_name: kafka-broker
    networks:
      - shipping_sovereign_network
    depends_on:
      - zookeeper
    ports:
      - "9092:9092"
    volumes:
      - kafka_data:/var/lib/kafka/data
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:29092,PLAINTEXT_HOST://localhost:9092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1


  # =========================
  # MIGRATION SERVICE (IMPORTANT 🔥)
  # =========================
  ledger-migrations:
    image: mcr.microsoft.com/dotnet/sdk:8.0
    container_name: ledger-migrations
    working_dir: /src
    volumes:
      - ./ledger-service/LedgerService:/src
    networks:
      - shipping_sovereign_network
    depends_on:
      postgres-ledger-db:
        condition: service_healthy
    entrypoint: ["bash", "-c"]
    command: >
      "dotnet tool install --global dotnet-ef &&
       export PATH=$PATH:/root/.dotnet/tools &&
       dotnet ef database update"


  # =========================
  # SERVICES
  # =========================

  gateway-service:
    build:
      context: ./gateway-service/GatewayService
      dockerfile: Dockerfile
    container_name: gateway-service
    networks:
      - shipping_sovereign_network
    depends_on:
      - kafka
      - postgres-ledger-db
      - mongo-shipment-db
    ports:
      - "8080:8080"
    environment:
      ASPNETCORE_ENVIRONMENT: Production
      ASPNETCORE_URLS: http://+:8080
    restart: unless-stopped


  ledger-service:
    build:
      context: ./ledger-service/LedgerService
      dockerfile: Dockerfile
    container_name: ledger-service
    networks:
      - shipping_sovereign_network
    depends_on:
      postgres-ledger-db:
        condition: service_healthy
    ports:
      - "8081:8080"
    environment:
      ASPNETCORE_ENVIRONMENT: Production
      ASPNETCORE_URLS: http://+:8080
    restart: unless-stopped


  fraud-service:
    build:
      context: ./fraud-service/FraudService
      dockerfile: Dockerfile
    container_name: fraud-service
    networks:
      - shipping_sovereign_network
    depends_on:
      - kafka
      - redis-distributed-cache
      - postgres-ledger-db
    restart: unless-stopped


  orchestrator-service:
    build:
      context: ./orchestrator-service/OrchestratorService
      dockerfile: Dockerfile
    container_name: orchestrator-service
    networks:
      - shipping_sovereign_network
    depends_on:
      - kafka
      - redis-distributed-cache
      - postgres-ledger-db
    restart: unless-stopped


  intelligence-service:
    build:
      context: ./intelligence-service
      dockerfile: Dockerfile
    container_name: intelligence-service
    networks:
      - shipping_sovereign_network
    depends_on:
      - kafka
      - redis-distributed-cache
      - mongo-shipment-db
    ports:
      - "3001:3000"
    environment:
      PORT: 3000
    restart: unless-stopped


  message-broker-service:
    build:
      context: ./message-broker-service
      dockerfile: Dockerfile
    container_name: message-broker-service
    networks:
      - shipping_sovereign_network
    depends_on:
      - kafka
    ports:
      - "3002:3000"
    environment:
      PORT: 3000
    restart: unless-stopped


  shipment-service:
    build:
      context: ./shipment-service/shipment-app
      dockerfile: Dockerfile
    container_name: shipment-service
    networks:
      - shipping_sovereign_network
    depends_on:
      - kafka
      - mongo-shipment-db
    ports:
      - "3003:3000"
    environment:
      PORT: 3000
    restart: unless-stopped
✨ Architectural Benchmarks Summary
Zero-Coupling Database Access: Microservices communicate state exclusively down internal message fabrics; no operational service directly probes a foreign store.

Boot Failure Mitigation: Strict service_healthy parameters prevent upstream apps from running and crashing due to storage initialization lag.

Optimized Fault Isolation: A failure inside the dynamic shipment-service node does not prevent the ledger-service from maintaining legal double-entry balances globally.

<h1 align="center">
  <br>
  <img src="https://img.shields.io/badge/Warehouse-Management%20System-0d1117?style=for-the-badge&labelColor=1a1d23" alt="WMS">
  <br><br>
  Warehouse Management System
  <br>
</h1>

<p align="center">
  <strong>A B2B SaaS platform for warehouse space rental and logistics</strong>
  <br>
  <sub>Warehouse owners list storage facilities with configurable zones and rooms. Business customers browse, book space, store goods, and request deliveries.</sub>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Java-25-ED8B00?style=flat-square&logo=openjdk&logoColor=white" alt="Java 25">
  <img src="https://img.shields.io/badge/Spring%20Boot-4-6DB33F?style=flat-square&logo=springboot&logoColor=white" alt="Spring Boot 4">
  <img src="https://img.shields.io/badge/PostgreSQL-18-4169E1?style=flat-square&logo=postgresql&logoColor=white" alt="PostgreSQL 18">
  <img src="https://img.shields.io/badge/Kafka-4-231F20?style=flat-square&logo=apachekafka&logoColor=white" alt="Kafka 4">
  <img src="https://img.shields.io/badge/Redis-8-DC382D?style=flat-square&logo=redis&logoColor=white" alt="Redis 8">
  <img src="https://img.shields.io/badge/Stripe-Payments-635BFF?style=flat-square&logo=stripe&logoColor=white" alt="Stripe">
  <img src="https://img.shields.io/badge/Docker-Compose-2496ED?style=flat-square&logo=docker&logoColor=white" alt="Docker">
</p>

<p align="center">
  <a href="https://Warehouse-Managament-System.github.io/WarehouseManagamentSystem/"><strong>Architecture Docs</strong></a> &nbsp;&middot;&nbsp;
  <a href="https://Warehouse-Managament-System.github.io/WarehouseManagamentSystem/c4.html"><strong>C4 Model</strong></a> &nbsp;&middot;&nbsp;
  <a href="https://Warehouse-Managament-System.github.io/WarehouseManagamentSystem/api.html"><strong>API Reference</strong></a> &nbsp;&middot;&nbsp;
  <a href="https://Warehouse-Managament-System.github.io/WarehouseManagamentSystem/database.html"><strong>Database Design</strong></a> &nbsp;&middot;&nbsp;
  <a href="https://Warehouse-Managament-System.github.io/WarehouseManagamentSystem/kafka.html"><strong>Kafka Events</strong></a>
</p>

---

## Architecture

**7 microservices** &nbsp;|&nbsp; **7 databases** &nbsp;|&nbsp; **38 tables** &nbsp;|&nbsp; **~60 REST endpoints** &nbsp;|&nbsp; **16 Kafka event types**

```
                                    ┌──────────────────────────────┐
                                    │       Spring Cloud Gateway   │
                                    │           (port 8080)        │
                                    └──────────────┬───────────────┘
                                                   │
                    ┌──────────────────────────────────────────────────────────┐
                    │                      Eureka Service Discovery            │
                    └──────────────────────────────────────────────────────────┘
                                                   │
         ┌──────────┬──────────┬──────────┬────────┴──┬──────────┬──────────┐
         │          │          │          │           │          │          │
    ┌────▼───┐ ┌────▼───┐ ┌───▼────┐ ┌───▼───┐ ┌────▼────┐ ┌───▼───┐ ┌───▼─────┐
    │Identity│ │  Ware- │ │Booking │ │ Goods │ │Delivery │ │Billing│ │Platform │
    │  :8081 │ │ house  │ │  :8083 │ │ :8084 │ │  :8085  │ │ :8086 │ │  :8087  │
    └───┬────┘ │  :8082 │ └───┬────┘ └──┬────┘ └────┬────┘ └──┬────┘ └────┬────┘
        │      └───┬────┘     │         │           │         │           │
    ┌───▼───┐  ┌───▼───┐  ┌──▼──┐  ┌───▼──┐  ┌────▼───┐  ┌──▼──┐  ┌────▼───┐
    │  DB   │  │  DB   │  │ DB  │  │  DB  │  │   DB   │  │ DB  │  │   DB   │
    └───────┘  └───────┘  └──┬──┘  └──────┘  └────────┘  └─────┘  └────────┘
                             │
                          ┌──▼───┐
                          │Redis │  ← Booking availability locks
                          └──────┘
         ┌─────────────────────────────────────────────────────────────────┐
         │                    Apache Kafka + Debezium CDC                  │
         │           Transactional Outbox → Event-Driven Communication     │
         └─────────────────────────────────────────────────────────────────┘
```

Each service owns its database — **no shared database access**. Services communicate asynchronously through Kafka events (via Transactional Outbox + Debezium CDC) and synchronously through REST calls (via OpenFeign + Resilience4j circuit breakers).

## Services

| Service | Port | Database | What it does |
|:--------|:----:|:--------:|:-------------|
| **Identity** | 8081 | `identity_db` | Auth (JWT), users, role-based profiles, Super Admin operations |
| **Warehouse** | 8082 | `warehouse_db` | Warehouses, zones, rooms, categories, Excel blueprint import |
| **Booking** | 8083 | `booking_db` | Booking lifecycle, Redis availability locks, expiry tracking |
| **Goods** | 8084 | `goods_db` | Goods Excel import, items, warehouse receipts, discrepancy handling |
| **Delivery** | 8085 | `delivery_db` | Delivery requests, picking, agent claims, shipment tracking |
| **Billing** | 8086 | `billing_db` | Invoices, Stripe Checkout + Webhooks, credit notes |
| **Platform** | 8087 | `platform_db` | Notifications, audit logs, scheduled batch jobs (Spring Batch + Quartz) |

**Infrastructure:** API Gateway (8080) &middot; Eureka (8761) &middot; Config Server (8888) &middot; Zipkin (9411)

## User Roles

```
SUPER ADMIN ──── Reviews and approves warehouse owner registrations

WAREHOUSE OWNER ──── Registers warehouses, configures zones/rooms via Excel,
                     manages staff and delivery agents, approves bookings and goods

CUSTOMER ──────── Browses warehouses, books storage space, uploads goods lists,
                  requests deliveries, pays invoices via Stripe

STAFF ─────────── Receives goods at warehouse, picks and packs orders,
                  notifies delivery agents when ready

DELIVERY AGENT ── Claims deliveries (first-come-first-served), picks up goods,
                  updates shipment checkpoints for customer tracking
```

## Event-Driven Communication

All inter-service async communication flows through **Apache Kafka** using the **Transactional Outbox** pattern:

```
Business logic writes data + outbox event (same DB transaction)
        │
        ▼
Debezium CDC watches outbox_events table via PostgreSQL WAL
        │
        ▼
Debezium publishes event to Kafka topic
        │
        ▼
Consumer(s) process event (idempotent — deduplication on event ID)
        │
        ▼ (on failure after 3 retries)
Dead Letter Topic ({topic-name}.DLT)
```

**16 event types** across booking, goods, delivery, billing, and identity domains — including `booking.confirmed`, `payment.success`, `delivery.claimed`, `goods.discrepancy`, and more.

## Tech Stack

| Layer | Technology |
|:------|:-----------|
| Language | Java 25 |
| Framework | Spring Boot 4, Spring Cloud 2025.x |
| Build | Gradle (Kotlin DSL) |
| Database | PostgreSQL 18, Spring Data JPA, Flyway |
| Messaging | Apache Kafka 4, Spring Kafka, Debezium CDC |
| Cache | Redis 8, Spring Data Redis |
| Security | Spring Security 7, JWT (jjwt) |
| Payments | Stripe Java SDK (Checkout + Webhooks) |
| Excel | Apache POI (import/export) |
| Resilience | Resilience4j (circuit breaker, retry, time limiter) |
| Service Comm | OpenFeign, Spring Cloud LoadBalancer |
| Tracing | Micrometer Tracing, Zipkin |
| Batch | Spring Batch 6, Quartz Scheduler |
| Containers | Docker, Docker Compose |

## Database Design

**7 isolated databases** &middot; **38 tables** &middot; UUID primary keys &middot; TIMESTAMPTZ everywhere &middot; Transactional Outbox per service

| Database | Tables | Key Entities |
|:---------|:------:|:-------------|
| `identity_db` | 5 + outbox | users, warehouse_owner_profiles, customer_profiles, staff_profiles, delivery_agent_profiles |
| `warehouse_db` | 9 + outbox | warehouses, zones, rooms, categories, zone_availabilities, images, Excel imports |
| `booking_db` | 2 + outbox | bookings (discriminated union: ROOM/ZONE/WAREHOUSE), expiry notifications |
| `goods_db` | 4 + outbox | goods_excel_imports, goods_items, goods_receipts, receipt_items |
| `delivery_db` | 5 + outbox | delivery_requests, request_items, notifications, shipments, checkpoints |
| `billing_db` | 4 + outbox | invoices (BOOKING/DELIVERY), invoice_items, payments, credit_notes |
| `platform_db` | 2 + outbox | notifications (15 types), audit_logs (immutable, JSONB diffs) |

## Documentation

Full architecture documentation is hosted via **GitHub Pages**:

| Page | Description |
|:-----|:------------|
| [Overview](https://Warehouse-Managament-System.github.io/WarehouseManagamentSystem/) | Architecture overview, services, tech stack |
| [C4 Architecture](https://Warehouse-Managament-System.github.io/WarehouseManagamentSystem/c4.html) | Interactive C4 model — L1 Context, L2 Container, L3 Component, L4 Code |
| [Database](https://Warehouse-Managament-System.github.io/WarehouseManagamentSystem/database.html) | Design decisions, 38 tables across 7 databases, entity schemas |
| [API](https://Warehouse-Managament-System.github.io/WarehouseManagamentSystem/api.html) | All ~60 REST endpoints grouped by service and role |
| [Kafka Events](https://Warehouse-Managament-System.github.io/WarehouseManagamentSystem/kafka.html) | 16 event types, payload contracts, consumer groups, DLT |

## Repository

| Repo | Description |
|:-----|:------------|
| [WarehouseManagamentSystem](https://github.com/Warehouse-Managament-System/WarehouseManagamentSystem) | Documentation, class diagrams, database schema, GitHub Pages site |

---

<p align="center">
  <sub>Built with Spring Boot, event-driven architecture, and database-per-service isolation.</sub>
</p>

# Project Vision — Finance Tracker

## Purpose

Finance Tracker is a personal side-project with two goals:

1. **Practical use** — give me a single place to monitor my stock holdings, bank investment funds, and day-to-day expenses.
2. **Skills development** — deliberately over-engineer the architecture so that every component is a learning exercise: event-driven design, microservices, message queues, containerisation, and eventually Kubernetes.

The implementation complexity should stay low (no premature optimisation), but the architectural complexity should be deliberately high.

---

## Core domains

### 1. Stocks Service
Tracks equity investments.

- CRUD for holdings (ticker, quantity, average buy price)
- Periodically fetches current prices from a market-data API (e.g. Alpha Vantage, Yahoo Finance)
- Calculates unrealised P&L, portfolio weight per holding
- Publishes `portfolio.updated` events to the queue whenever prices are refreshed

### 2. Investment Funds Service
Tracks bank-managed investment funds (e.g. index funds, ETFs held through a bank platform).

- CRUD for fund positions (fund name/ISIN, units held, purchase NAV)
- Manual or API-driven NAV updates
- Calculates current value and return
- Publishes `funds.updated` events

### 3. Expenses Service
Tracks personal spending.

- CRUD for expense entries (amount, category, date, notes)
- Category management
- Monthly/weekly aggregations and trends
- Publishes `expenses.recorded` events (useful for future budgeting rules)

### 4. Web UI
A single-page application that is the user-facing layer.

- Dashboard: combined net-worth overview (stocks + funds + cash estimate)
- Per-domain pages for drilldown
- Real-time (or near-real-time) updates via WebSocket or Server-Sent Events connected to the API Gateway
- No business logic — purely presentational

---

## Architecture

### Microservices

Each domain is a standalone service with its own:
- Codebase and deployment unit
- Database (PostgreSQL schema or instance)
- REST API consumed by the Gateway

This is over-engineered for a single user, but it is the point — it forces proper service boundary design, independent deployability, and database-per-service discipline.

### API Gateway

A thin gateway (Node.js) that:
- Exposes a unified API or GraphQL schema to the front-end
- Handles authentication (JWT)
- Routes requests to the appropriate service
- Aggregates responses where the UI needs data from multiple services

### Message Queue

A central message broker (RabbitMQ initially, with the option to migrate to Kafka) that:
- Decouples services — services publish events without knowing who consumes them
- Enables future consumers (notification service, report generator, budgeting engine) to be added without touching existing services
- Provides a history/audit trail of domain events

**Key events (initial set):**

| Event | Publisher | Consumer(s) |
|---|---|---|
| `portfolio.updated` | Stocks Service | UI push, future: alerting |
| `funds.updated` | Funds Service | UI push, future: aggregator |
| `expenses.recorded` | Expenses Service | future: budgeting, analytics |
| `report.requested` | UI / scheduled job | future: report generator |

### Event-driven design (aspirational)

The long-term goal is a fully event-driven architecture where:
- Services only communicate via events (no direct service-to-service HTTP calls)
- State can be rebuilt from the event log (event sourcing, at least for the portfolio services)
- A separate read-model / query service can materialise aggregated views

This will be introduced incrementally — the first version will use direct HTTP between the gateway and services, with the queue used only for notifications/updates to the UI.

---

## Infrastructure

| Phase | Target |
|---|---|
| Phase 1 | Docker Compose on a single machine / laptop |
| Phase 2 | Docker Compose on a cheap VPS or home server |
| Phase 3 | Kubernetes (k3s or managed) — for the learning experience |

Infrastructure-as-code (Terraform or Pulumi) will be introduced at Phase 2.

---

## Architecture Decision Records (ADRs)

Significant decisions will be recorded in `docs/adr/`. Planned initial ADRs:

- `0001-microservices-over-monolith.md` — why microservices for a one-user app
- `0002-message-queue-choice.md` — RabbitMQ vs Kafka
- `0003-database-per-service.md` — rationale and isolation strategy
- `0004-api-gateway-pattern.md` — gateway vs BFF vs direct calls
- `0005-event-sourcing-scope.md` — where (if anywhere) to apply event sourcing

---

## Non-goals (for now)

- Multi-user support / SaaS
- Mobile app
- Automated trading
- Integration with bank APIs (expenses will be manual or CSV import initially)
- High availability / disaster recovery beyond basic backups

---

## Success criteria

- I can see my total net worth (stocks + funds) on a single page
- I can log an expense in under 30 seconds
- Adding a new service or consumer requires no changes to existing services
- The entire stack starts with `docker compose up`
- The project serves as a portfolio piece demonstrating distributed systems knowledge

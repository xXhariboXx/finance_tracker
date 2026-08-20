# Finance Tracker

A personal finance tracking web application built as a side-project for skills development. The app is designed with a microservices architecture and an event-driven backbone — intentionally more robust than strictly necessary, as an exercise in building production-grade distributed systems.

---

## What it does

| Domain | Description |
|---|---|
| **Stock Market** | Track stock holdings, prices, and portfolio performance |
| **Investment Funds** | Monitor bank-managed investment funds and their NAV over time |
| **Expenses** | Log and categorise personal expenses, view spending trends |
| **Web UI** | Single unified dashboard to view all three domains |

---

## Architecture overview

```
┌─────────────────────────────────────────────────────────┐
│                        Web UI                           │
│               (React / Next.js SPA)                     │
└───────────────────────┬─────────────────────────────────┘
                        │ REST / GraphQL / WebSocket
          ┌─────────────▼─────────────┐
          │        API Gateway        │
          └──┬──────────┬──────────┬──┘
             │          │          │
    ┌────────▼──┐ ┌─────▼────┐ ┌──▼──────────┐
    │  Stocks   │ │  Funds   │ │  Expenses   │
    │  Service  │ │  Service │ │  Service    │
    └────────┬──┘ └─────┬────┘ └──┬──────────┘
             │          │         │
             └──────────▼─────────┘
                   Message Queue
                 (RabbitMQ / Kafka)
                        │
             ┌──────────▼──────────┐
             │   Event Consumers   │
             │ (notifications,     │
             │  aggregations, etc) │
             └─────────────────────┘
```

Each service owns its own database. Services communicate asynchronously via the message queue for cross-domain events (e.g. "portfolio rebalance needed", "monthly report ready").

See [VISION.md](./VISION.md) for the full architectural rationale and roadmap.

---

## Repository structure _(planned)_

```
finance_tracker/
├── services/
│   ├── stocks/          # Stock market service
│   ├── funds/           # Investment funds service
│   ├── expenses/        # Expenses service
│   └── gateway/         # API Gateway
├── web/                 # Front-end application
├── infra/               # Docker Compose, Kubernetes manifests, etc.
└── docs/                # Architecture decision records (ADRs)
```

---

## Getting started _(coming soon)_

Prerequisites: Docker, Docker Compose.

```bash
git clone https://github.com/xXhariboXx/finance_tracker.git
cd finance_tracker
docker compose up
```

The web UI will be available at `http://localhost:3000`.

---

## Tech stack _(provisional)_

| Layer | Technology |
|---|---|
| Front-end | React / Next.js |
| API Gateway | Node.js (Express or Fastify) |
| Services | Node.js or Python (per service) |
| Message Queue | RabbitMQ or Apache Kafka |
| Databases | PostgreSQL (per service) |
| Infrastructure | Docker Compose → Kubernetes |

---

## Contributing

This is a personal side-project. Feel free to open issues or discussions if you have ideas or spot something interesting.

---

## Licence

MIT
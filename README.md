# Exchange Project Overview

This repository contains the full **Limit Order Book (LOB) + Exchange** stack for a low-latency trading environment, including the Rust exchange core, the Next.js trading client, end-to-end docs, and deployment tooling.

## What this repo includes

- `exchange/` — Rust exchange core service.
  - Matching engine and in-memory orderbook.
  - REST + WebSocket APIs for market data, order entry, user state, and admin controls.
  - User API key auth, admin bearer auth, rate limiting, checkpoint persistence, and competition operator workflows.
  - Market-data enrichment, raw order-event feeds, and delta-snapshot resync semantics.

- `client/` — Next.js low-latency exchange client template.
  - Browser trading UI, login flow, admin console, and integration with the exchange API.
  - ECS-ready production Docker setup and environment-driven backend configuration.

- `docs/` — Public API and protocol documentation.
  - Mintlify-powered docs for REST and WebSocket routes and client integration.

- `internal-docs/` — Internal architecture, deployment, and competition notes.
  - Operator guides, system design notes, and deployment-specific runbooks.

- `infra/` — Deployment infrastructure.
  - ECS client deployment stack, certificate automation, and environment setup scripts.

- `tools/` — Utilities and stress-test clients.
  - Load bench, stress trading bots, user provisioning scripts, and market-order client samples.

## Core concepts

### Limit Order Book (LOB)

The LOB is the heart of the exchange.

- Accepts order submissions, amendments, and cancellations.
- Maintains per-market bid/ask depth, open orders, and trade matching.
- Produces canonical order events used by all downstream feeds and client views.
- Exposes public and authenticated market data over WebSocket:
  - `l2` aggregated depth snapshots and deltas.
  - `l3` raw order-level snapshots and canonical order events.
- Uses in-memory matching with periodic checkpoint persistence for restart resilience.

### Exchange service

The Rust exchange service exposes:

- REST APIs for market discovery, user state, balances, positions, open orders, fills, and operator controls.
- WebSocket for real-time market data, authenticated trading, and user event delivery.
- Admin controls for trading lifecycle, market configuration, user provisioning, and leaderboard snapshots.
- Simple competition-focused auth model using assigned API keys and admin bearer tokens.

## How the pieces fit together

- The `exchange/` service is the backend execution engine and market-data source.
- The `client/` application is the trading UI that consumes exchange APIs and WS feeds.
- `docs/` defines the public API contract that both backend and client can rely on.
- `infra/` provides deployment support for ECS and production-ready hosting.
- `tools/` supports load testing, stress testing, and test account provisioning.

## Getting started

### Run the exchange service locally

```bash
cd exchange
cargo run --bin exchange
```

Optionally run the market-data service locally in split-process mode:

```bash
export MARKET_DATA_SERVICE_SOCKET=/tmp/exchange-marketdata.sock
cargo run --bin market-data-service &
cargo run --bin exchange
```

Then open:

- `http://localhost:8080/health`
- `http://localhost:8080/docs`

### Run the client locally

```bash
cd client
npm install
npm run dev
```

Configure backend endpoints with environment variables:

- `EXCHANGE_HTTP_URL`
- `NEXT_PUBLIC_EXCHANGE_HTTP_URL`
- `NEXT_PUBLIC_EXCHANGE_WS_URL`

## Docs and architecture

- Public API docs: `docs/`
- Internal deployment and architecture notes: `internal-docs/`

## Deployment notes

- Client ECS deployment lives under `infra/client-ecs/`.
- The exchange service currently targets an EC2-based deployment with checkpoint persistence.
- Keep environment and deployment secrets externalized in AWS Secrets Manager or Parameter Store.

## Repo purpose

This repository is designed as a complete LOB + exchange implementation for a trading competition or internal low-latency exchange product. It includes both the exchange execution core and the trading client, plus the docs, infra, and tooling needed to operate, test, and deploy the system.

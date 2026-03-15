# 2048 BOOSTED — Technical Stack Recommendation

**Version:** 1.0 | **Date:** 2026-03-10

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────┐
│                      CLIENTS                            │
│  ┌──────────┐  ┌──────────┐  ┌───────────────────────┐ │
│  │ React    │  │ React    │  │ Progressive Web App   │ │
│  │ Native   │  │ Native   │  │ (Next.js)             │ │
│  │ (iOS)    │  │ (Android)│  │                       │ │
│  └────┬─────┘  └────┬─────┘  └──────────┬────────────┘ │
│       └──────────────┴──────────────────┘               │
│                      │                                  │
│            REST API + WebSocket                         │
└──────────────────────┬──────────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────────┐
│                   BACKEND                               │
│  ┌────────────────┐  ┌────────────────┐                 │
│  │ API Gateway    │  │ WebSocket GW   │                 │
│  │ (Express.js)   │  │ (Socket.io)    │                 │
│  └───────┬────────┘  └───────┬────────┘                 │
│          │                   │                          │
│  ┌───────▼───────────────────▼────────┐                 │
│  │         Service Layer              │                 │
│  │  ┌─────────┐ ┌──────┐ ┌────────┐  │                 │
│  │  │ Match-  │ │Bet-  │ │Game    │  │                 │
│  │  │ making  │ │ting  │ │Engine  │  │                 │
│  │  └────┬────┘ └──┬───┘ └───┬────┘  │                 │
│  └───────┼─────────┼─────────┼───────┘                 │
│          │         │         │                          │
│  ┌───────▼─────────▼─────────▼───────┐                 │
│  │          Data Layer               │                 │
│  │  ┌──────────┐  ┌──────────────┐   │                 │
│  │  │ Redis    │  │ PostgreSQL   │   │                 │
│  │  │ (cache,  │  │ (persistent) │   │                 │
│  │  │  pub/sub)│  │              │   │                 │
│  │  └──────────┘  └──────────────┘   │                 │
│  └───────────────────────────────────┘                 │
└─────────────────────────────────────────────────────────┘
```

---

## Stack Details

### Client

| Layer | Technology | Justification |
|-------|-----------|---------------|
| **Mobile** | React Native + Expo | Single codebase for iOS/Android, fast iteration, OTA updates |
| **Web** | Next.js (React) | SSR for SEO (landing pages), shared component library with mobile |
| **Game Rendering** | `react-native-canvas` / HTML5 Canvas | Lightweight 2D rendering, no heavy engine needed for tile-based puzzles |
| **State Management** | Zustand | Minimal boilerplate, performant, great for real-time state |
| **Animations** | React Native Reanimated + Lottie | 60fps animations for tile merges, combos, and attack effects |

### Server

| Layer | Technology | Justification |
|-------|-----------|---------------|
| **Runtime** | Node.js 22 LTS | Non-blocking I/O ideal for real-time; massive ecosystem |
| **API Framework** | Express.js | Mature, extensive middleware support |
| **Real-Time** | Socket.io | Reliable WebSocket with fallback, rooms, namespaces, auto-reconnect |
| **Authentication** | Firebase Auth + JWT | Social login (Google/Apple), secure session tokens |
| **Task Queue** | BullMQ (Redis-backed) | Deferred jobs: reward distribution, matchmaking retries, analytics |

### Database

| Store | Technology | Purpose |
|-------|-----------|---------|
| **Primary DB** | PostgreSQL 16 | Player profiles, transaction ledger, leaderboards, match history |
| **Cache / Pub-Sub** | Redis 7 | Matchmaking queues, live game state, session cache, cross-server pub/sub |
| **Object Storage** | AWS S3 / Cloudflare R2 | Skin assets, avatar images, loot box assets |

> Indexes will be created on player ELO ratings and match timestamps to ensure fast leaderboard queries and profile loads.

### Infrastructure

| Concern | Technology | Notes |
|---------|-----------|-------|
| **Containerization** | Docker + docker-compose | Reproducible local dev and CI |
| **Orchestration** | Kubernetes (EKS/GKE) | Auto-scale game servers by demand |
| **CI/CD** | GitHub Actions | Lint → Test → Build → Deploy pipeline |
| **Monitoring** | Grafana + Prometheus | Server metrics, match latency, error rates |
| **Logging** | ELK Stack or Datadog | Centralized logs for debugging |
| **Payments** | RevenueCat (mobile IAP) + Stripe (web) | Unified subscription/purchase management |

### Anti-Cheat

- **Server-authoritative game loop** — client sends move direction, server computes new state.
- **Move validation** — server rejects impossible moves.
- **Rate limiting** — max 2 moves/second to prevent botting.
- **Replay audit log** — full move history stored for dispute resolution.

---

## Why NOT Unity/Godot?

| Factor | Unity/Godot | React Native + Node.js |
|--------|------------|----------------------|
| Development speed | Slower for UI-heavy games | Faster iteration, hot reload |
| Multiplayer infra | Must build or buy (Photon) | Node.js + Socket.io native |
| Web support | WebGL builds are heavy | Native web via Next.js |
| Team skills | Requires C#/GDScript | JavaScript/TypeScript everywhere |
| Monetization | Plugin-dependent | Direct API integrations |
| App size | 30–80 MB minimum | 10–20 MB |

> **Verdict:** For a tile-based puzzle with heavy server-side logic (betting, matchmaking, anti-cheat), a web-native stack is leaner, faster to ship, and cheaper to scale.

---

*End of Tech Stack v1.0*

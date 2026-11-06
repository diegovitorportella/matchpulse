# ⚽ MatchPulse

> Full-stack live football dashboard: scores, match timelines and stats in real time, with its own API, user accounts and synced favorites.

![Status](https://img.shields.io/badge/status-planning-blue)
![Angular](https://img.shields.io/badge/Angular-21-dd0031)
![NestJS](https://img.shields.io/badge/NestJS-API-e0234e)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-data-336791)
![TypeScript](https://img.shields.io/badge/TypeScript-5.x-3178c6)
![License](https://img.shields.io/badge/license-MIT-green)

---

## 📑 Table of Contents

1. [Name and Description](#1-name-and-description)
2. [Goal and Problem Solved](#2-goal-and-problem-solved)
3. [Target Audience](#3-target-audience)
4. [MVP Features](#4-mvp-features)
5. [Future Features](#5-future-features)
6. [Tech Stack and Rationale](#6-tech-stack-and-rationale)
7. [Architecture and Organization](#7-architecture-and-organization)
8. [Initial Folder Structure](#8-initial-folder-structure)
9. [Step-by-Step Development Plan](#9-step-by-step-development-plan)
10. [Best Practices and Code Standards](#10-best-practices-and-code-standards)
11. [Technical Challenges and Skills Developed](#11-technical-challenges-and-skills-developed)
12. [Future Evolution](#12-future-evolution)

---

## 1. Name and Description

**MatchPulse** is a **full-stack** web application that gathers the day's football matches in a single dashboard. It shows automatically updated scores, a match event timeline (goals, cards, substitutions) and visual stats (possession, shots, corners).

The project has two parts:

- **Front-end (Angular):** reactive dashboard, filters, charts and favorites.
- **Back-end (NestJS + PostgreSQL):** queries the public football API **once for all users**, stores the data, authenticates users and pushes live updates to clients.

The name refers to the "pulse" of the game: the system "beats" at regular intervals and pushes news to the user.

## 2. Goal and Problem Solved

### Problem
Football fans often follow several matches at once and have to switch between apps and sites full of ads. On top of that, public football APIs have **rate limits**: if every user queried the API directly, the limit would be reached quickly and the API key would be exposed.

### Goal
- Deliver a **fast, clean, focused** experience for the day's matches.
- Provide an **own back-end** that protects the key, caches data and respects the source's limits.
- Sync **favorites across devices** through user accounts.

### Learning goal
Practice **RxJS and NgRx** on the front-end, and **REST APIs, authentication, relational databases, scheduled jobs and real-time delivery** on the back-end, tying everything into a complete product.

## 3. Target Audience

- **Football fans** who want to follow several matches without distractions.
- **Betting-pool participants** who need quick results.
- **Recruiters and developers**, as a practical full-stack TypeScript showcase.

## 4. MVP Features

### Front-end
| # | Feature | Description |
|---|---|---|
| 1 | **Today's matches list** | Matches grouped by competition, with status and score. |
| 2 | **Real-time updates** | Receives server events; reconnects and resyncs if the connection drops. |
| 3 | **Match details** | Score, chronological timeline and stats charts. |
| 4 | **Sign-up and login** | Own account, persistent session and automatic token refresh. |
| 5 | **Synced favorites** | Favorite teams and matches saved to the account. |
| 6 | **Filters and date navigation** | By competition, status and day, reflected in the URL. |
| 7 | **UI states** | Skeletons, empty state, error with retry, server-offline mode. |
| 8 | **Light/dark theme, responsive** | Respects the system preference. |

### Back-end
| # | Feature | Description |
|---|---|---|
| 9 | **Scheduled ingestion** | A job queries the external API at intervals (more often during live matches) and writes to the database. |
| 10 | **Own REST API** | Matches, competitions, details, favorites and users. |
| 11 | **Real time via SSE** | Event channel announcing goals, status changes and score updates. |
| 12 | **Authentication** | Sign-up, login, short-lived access token and refresh token. |
| 13 | **Cache and protection** | In-memory cache, per-client rate limiting and input validation. |
| 14 | **API documentation** | Browsable OpenAPI specification. |
| 15 | **Mock mode** | Simulated data source to develop and demo without the external API. |

**Out of the MVP:** push notifications, social login, admin panel.

## 5. Future Features

- Goal notifications for favorite teams.
- League table and team comparison.
- Multi-view: 2 to 4 matches side by side.
- Social login.
- Distributed cache and job queue.
- Internationalization (pt-BR, en, es).
- Installable PWA.

## 6. Tech Stack and Rationale

| Layer | Technology | Rationale |
|---|---|---|
| Front | **Angular (current stable) + strict TypeScript** | Opinionated structure, great for projects that grow. |
| Front | **RxJS** | Reconnection, cancellation, combining sources and SSE as a stream. |
| Front | **NgRx (Store, Effects, Entity, Router Store)** | Normalized match collection and derived state via selectors. |
| Front | **Chart.js** | Lightweight stats charts. |
| Front | **Angular CDK + SCSS with tokens** | Accessibility and theming without a heavy UI library. |
| Back | **NestJS (TypeScript)** | Modules and dependency injection similar to Angular; shortens the learning curve and ships validation, guards and scheduling. |
| Back | **PostgreSQL** | Relational data (competition → match → event) with integrity. |
| Back | **ORM with migrations** | Versioned schema and typed queries. |
| Back | **JWT (access + refresh)** | Stateless authentication, industry standard; strongly hashed passwords. |
| Back | **SSE (Server-Sent Events)** | Enough for server → client traffic, simpler than WebSocket. |
| Back | **NestJS task scheduler** | Periodic ingestion without extra infrastructure. |
| Back | **OpenAPI** | Documented, testable contract. |
| Infra | **Docker Compose** | Database and API start with one command, reproducible environment. |
| Quality | **Jest (front and back) + Supertest** | Unit and API tests with the same tool. |
| Quality | **ESLint + Prettier + CI** | Consistency and automated feedback. |
| Shared | **`contracts` (TypeScript types)** | Front and back share the same data models. |

> **Decision:** the cache starts **in memory**, without Redis, to avoid complexity. Swapping it in is listed as a future evolution.

## 7. Architecture and Organization

### 7.1 Overview

```
 External football API             (rate limited)
        │  (only the back-end queries it)
        ▼
┌──────────────────────── BACK-END (NestJS) ────────────────────────┐
│  Scheduled ingestion → Adapter → Matches service → PostgreSQL      │
│                                   │                                │
│  Auth (JWT) · Favorites · REST API · Event emitter (SSE)           │
└───────────────┬───────────────────────────────┬───────────────────┘
                │ REST                          │ SSE
                ▼                               ▼
┌──────────────────────── FRONT-END (Angular) ──────────────────────┐
│  Interceptors (token, errors) → NgRx (effects, store, selectors)   │
│  → OnPush pages and components                                     │
└───────────────────────────────────────────────────────────────────┘
```

### 7.2 Back-end principles
- **Modules per domain:** authentication, users, matches, competitions, favorites, ingestion, real-time events.
- **Layers:** controller (HTTP) → service (business rules) → repository (data). Controllers hold no business logic.
- **External source adapter:** the public API response becomes the internal model. Switching providers only affects the adapter.
- **Validated DTOs:** every input is validated and typed; nothing from the client is trusted.
- **Idempotent ingestion:** reprocessing the same response does not duplicate data.
- **Failure tolerance:** if the external source goes down, the API keeps serving the last saved state.
- **Basic security:** password hashing, rate limiting, restricted CORS, secrets in environment variables, security headers.

### 7.3 Front-end principles
- **Feature folders** with lazy loading; "smart" pages and presentational components.
- **Side effects only in Effects:** components never call HTTP directly.
- **Derived state via selectors:** filters and grouping are computed.
- **SSE as an Observable:** the stream feeds the store; on reconnect, it fetches the current state to cover what was missed.

### 7.4 API contract (summary)

| Resource | Operations |
|---|---|
| Authentication | register, log in, refresh token, log out |
| Matches | list by date and filters, get details (events and stats) |
| Competitions | list |
| Favorites | list, add, remove (authenticated) |
| Real time | SSE channel for match events |
| Health | API and source status check |

### 7.5 Data model (conceptual)
**User** · **Competition** · **Team** · **Match** (status, score, kickoff time) · **Match event** (type, minute, player) · **Match stats** · **Favorite** (user ↔ team/match) · **Refresh token**.

## 8. Initial Folder Structure

```
matchday-pulse/
├── README.md
├── package.json                  # workspaces: frontend, backend, contracts
├── docker-compose                # PostgreSQL + API
├── .github/                      # CI: lint, tests, build
├── contracts/                    # shared types (Match, Event, Stats, DTOs)
├── backend/
│   ├── .env.example
│   ├── migrations/
│   ├── mocks/                    # simulated external API responses
│   └── src/
│       ├── main
│       ├── config/               # validated environment variables
│       ├── common/               # error filters, guards, interceptors, utilities
│       └── modules/
│           ├── auth/
│           ├── users/
│           ├── competitions/
│           ├── matches/
│           ├── favorites/
│           ├── ingestion/        # scheduled job + adapter + source (real | mock)
│           └── realtime/         # SSE emitter
└── frontend/
    └── src/
        ├── styles/               # tokens and themes
        ├── environments/
        └── app/
            ├── core/             # auth, interceptors, SSE, theme, models
            ├── shared/           # UI, pipes, utilities
            ├── store/            # NgRx root
            └── features/
                ├── auth/
                ├── matches/
                ├── match-details/
                └── favorites/
```

## 9. Step-by-Step Development Plan

Estimate for part-time work: **9 to 12 weeks**. Each phase ends with something working and tested.

### Phase 0 — Foundation (4 to 5 days)
- [ ] Create the monorepo with workspaces and the contracts package.
- [ ] Configure ESLint, Prettier, Jest and CI across all packages.
- [ ] Start PostgreSQL via Docker Compose.
- [ ] Define commit conventions and environment variables (`.env.example`).

### Phase 1 — Back-end base (1.5 weeks)
- [ ] Create the NestJS application with validated configuration and a health route.
- [ ] Model the database and create the first migrations.
- [ ] Implement competitions, teams and matches (read) with DTOs and validation.
- [ ] Document with OpenAPI.
- [ ] Service and API tests.

### Phase 2 — Data ingestion (1 to 1.5 weeks)
- [ ] Choose the external API; study limits, format and available fields.
- [ ] Build the **mock** source first, then the real one, behind a common interface.
- [ ] Build the adapter and the scheduled idempotent ingestion.
- [ ] Tune frequency: shorter with live matches, longer without.
- [ ] Test the adapter and behavior when the source fails.

### Phase 3 — Authentication and favorites (1.5 weeks)
- [ ] Sign-up and login with password hashing.
- [ ] Short-lived access token and refresh token, with revocation.
- [ ] Route guards and rate limiting.
- [ ] Favorites endpoints.
- [ ] Security tests (no token, expired token).

### Phase 4 — Front-end: matches list (1.5 weeks)
- [ ] Create the Angular project and design tokens.
- [ ] Configure NgRx and consume the API.
- [ ] Build the grouped list, with skeletons, empty and error states.
- [ ] Interceptors for token and error handling.

### Phase 5 — Real time (1 to 1.5 weeks)
- [ ] Build the SSE emitter on the back-end.
- [ ] Consume the channel as an Observable, with reconnection and resync.
- [ ] Highlight score changes.
- [ ] Test the flow with RxJS's test scheduler.

### Phase 6 — Details, filters and account (1.5 weeks)
- [ ] Details page with timeline and charts.
- [ ] Filters and date reflected in the URL.
- [ ] Sign-up and login screens; automatic token refresh.
- [ ] Favorites synced with the account.

### Phase 7 — Quality and release (1 to 1.5 weeks)
- [ ] Review accessibility, responsiveness and performance.
- [ ] Increase coverage on critical flows.
- [ ] Package with Docker and deploy front and API to free hosting.
- [ ] Final README with screenshots, GIF and diagram.

## 10. Best Practices and Code Standards

**General**
- Strict TypeScript, no `any`; contracts shared between front and back.
- Small commits (*Conventional Commits*); code names in English.
- **Never** commit secrets; use `.env.example` and environment variables.

**Back-end**
- Thin controllers; business rules in services; data access in repositories.
- Validate all input and standardize error responses.
- Versioned migrations; no manual database changes.
- Strongly hashed passwords; never log sensitive data.
- Structured logs with a request identifier.
- Tests: service unit tests, API integration tests (Supertest) and adapter tests.

**Front-end**
- `OnPush` on all components; standalone components and lazy loading.
- Actions as events; pure reducers; memoized selectors.
- Error handling inside the effects' inner stream.
- Every long-lived stream has a stop condition; cancel subscriptions.
- Accessibility from the start (semantic HTML, focus, contrast).

## 11. Technical Challenges and Skills Developed

| Challenge | What you learn |
|---|---|
| Scheduled, idempotent ingestion | Jobs, consistency, tolerance to external failures |
| Respecting a third-party API's limits | Caching, adaptive frequency, key protection |
| Authentication with token refresh | JWT, password hashing, guards, interceptors |
| SSE with reconnection and resync | Real time, advanced RxJS, state consistency |
| Relational modeling | Keys, indexes, migrations |
| Documented and tested API | OpenAPI, Supertest, contracts |
| Normalized state with NgRx Entity | Immutability, selectors, derived state |
| Shared types in a monorepo | Contracts, reducing front/back drift |
| Containerization | Docker Compose, reproducible environment |
| Basic web security | CORS, rate limiting, input validation |

## 12. Future Evolution

- **Redis** for distributed caching and ingestion queues.
- **WebSocket** if two-way communication becomes necessary.
- **Push notifications** for favorite teams' goals.
- **Social login** and password recovery.
- **Observability:** metrics, health dashboard and alerts.
- **End-to-end tests** (list → details → favorite) in CI.
- **Other sports**, reusing the adapter and source contract.
- **Roles and admin panel** to manage competitions.

---

## 📝 Final Notes

- This document is a **planning guide**; adjust the scope to your pace.
- Finish one **vertical slice** first (a single match, end to end) before expanding.
- After each completed phase, update the checkboxes and record what you learned.

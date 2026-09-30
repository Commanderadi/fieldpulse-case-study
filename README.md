# FieldPulse: Field Sales Force Automation (Case Study)

> **Live product:** [fieldpulse.in](https://fieldpulse.in) · Source is private; this repo documents the architecture and engineering decisions.

FieldPulse is a field sales force automation platform I **conceived and built solo** at ELETTRO, a manufacturing company. It started as an internal tool and was later **spun out as a SaaS product** for other distributors.

It replaces WhatsApp updates and spreadsheets with verified visit data. Managers see where their field officers are, what they did and what it cost, in real time.

---

## At a glance

| | |
|---|---|
| **Role** | Sole engineer: idea, schema, API, mobile, admin UI, infrastructure |
| **Backend** | NestJS · TypeORM · PostgreSQL · Redis · Socket.IO |
| **Mobile** | Flutter (Riverpod, Dio, go_router) |
| **Admin** | React dashboard with a live map |
| **Storage** | Cloudflare R2 (photos) |
| **Infra** | Self-hosted on Hetzner · nginx · Let's Encrypt · Docker · GitHub Actions CI/CD |
| **Scale of codebase** | ~173 REST endpoints · 26 database entities |

---

## The problem

Field sales teams at manufacturers and distributors report visits, travel and orders manually. Managers can't easily check whether a visit really happened, travel reimbursements are hard to audit, and reporting lags by days.

## What I built

**1. Flutter field app**
- **GPS-verified visit check-in:** a visit only counts if the officer is actually at the customer's location
- **Photo-verified odometer capture** at the start and end of each day, so travel claims come from evidence rather than self-reported numbers
- Visit logs, customer records and quotations from the field

**2. React admin dashboard**
- **Live map** of field officers, with location updates streamed over WebSockets (Socket.IO + Redis)
- **Fraud-detection view** that flags check-ins and travel that don't add up
- Visit, quotation and team management; reporting

**3. NestJS / PostgreSQL backend**
- ~173 REST endpoints over 26 entities (users, roles, customers, visits, odometer logs, quotations…)
- Role-based access control, JWT auth, rate limiting
- Designed for multiple tenants so it could be offered to other distributors

---

## Architecture

```
 Flutter app ──HTTPS──▶ nginx ──▶ NestJS API ──▶ PostgreSQL
      │                             │   │
      │  photos                     │   └──▶ Redis (pub/sub, caching)
      └────────────▶ Cloudflare R2  │
                                    └──Socket.IO──▶ React admin (live map)

 GitHub Actions ──build/test/deploy──▶ Hetzner (Docker)
```


---

## Engineering decisions

- **Self-hosting on Hetzner instead of a PaaS:** much lower cost for an always-on API with WebSockets, full control over nginx and TLS, and predictable pricing for a SaaS with small margins.
- **Redis between the API and Socket.IO:** location updates fan out to admin clients without polling the database.
- **Photos straight to object storage (R2):** the API never streams large uploads, which keeps it light and cheap.
- **Reliability patterns:** retry logic, idempotent handlers for async work and structured logging, so flaky mobile networks don't create duplicate visits.

## Outcome

- Went from an internal tool to a standalone SaaS product: **[fieldpulse.in](https://fieldpulse.in)**
- Replaced manual, unverifiable visit reporting with GPS- and photo-verified data

---

*Want a walkthrough? Reach me at [adityasingh3499@gmail.com](mailto:adityasingh3499@gmail.com) or on [LinkedIn](https://www.linkedin.com/in/aditya-singh-89b884189).*

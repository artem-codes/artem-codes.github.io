---
layout: page
title: "Microservices in Practice: Service Communication & Workflow Orchestration"
permalink: /log/microservices-communication/
tags: [saga, rabbitmq, cqrs, dotnet]
---

> This page is a technical annex to my résumé. It is aimed at a technical specialist who wants to see concrete architecture decisions rather than a list of technologies on a CV.

## Context

Since microservices communication is a hot topic, I decided to prepare an example of a distributed system. The names and domains are simplified, but the architecture reflects how I actually build things — which patterns I use, how services talk to each other, and what trade-offs come into play.

## Document Note Notification Schema

<iframe width="100%" height="600" src="https://miro.com/app/live-embed/uXjVN4NLetI=/?focusWidget=3458764673881747356&embedMode=view_only_without_ui&embedId=886571827000" frameborder="0" scrolling="no" allow="fullscreen; clipboard-read; clipboard-write" allowfullscreen></iframe>

### Flow Overview

1. **Admin** adds a note via `Docs.Editor.Web` → `command.document.note.add`
2. `Docs.Metadata.State` updates the document summary
3. `Docs.Notifications.Workflow` (**state machine**) triggers immediate notification + schedules digest/timeout — cancellable on early view
4. `Notifications.State` + `EventTicker` handle scheduled task execution
5. `Notifications.Dispatcher` enriches recipient/template → `command.notification.email.send`
6. `Notifications.SMTP` delivers email → `event.notification.sent` / `failed`
7. **Customer** opens link → `Docs.Viewer.Web` emits `event.document.note.viewed`, cancelling pending notifications

---

## Patterns in Action

### DDD — Bounded Contexts

Services are split by domain and subdomain:
Each context owns its data and communicates only through the bus.

### CQRS

Every message on the bus is strictly typed:

- **Command** — one consumer, expresses intent
- **Query** — one consumer,  request/reply pattern
- **Event** — many consumers, immutable fact, fan-out.

### State Machine

`Docs.Notifications.Workflow` is a Saga state machine where incoming events drive state transitions;

### Event-driven scheduling
`Notifications.EventTicker` emits periodic ticks; `Notifications.State` reacts by scanning for due tasks. Decouples schedule from execution.

> High-load scheduling is a topic that deserves its own article: [High-Load Database Patterns](/log/high-load-database/). A few considerations:
>
> - `Time-shift` — scheduled tasks are spread across a window (e.g. ±5 min jitter) to avoid spikes.
> - `Conditional suppression` — skip emission when recipients are in night hours (no point waking a queue if nobody will read the email).
> - `Partitioning` — tasks are partitioned so consumers process slices in parallel.
 
### Thin Web Gateways

`Docs.Editor.Web` / `Docs.Viewer.Web` handle only auth, validation, and routing — no domain logic.

### Idempotency

`Docs.Viewer.Web` is the most obvious source of duplicates, but we have to be ready to handle duplicates across all services. All downstream consumers should be **idempotent** — processing the same event twice produces no side effects.

### Pipes and Filters — Processing Pipeline

Notification delivery is a linear pipeline where each step performs a single transformation:

```
Notification → Recipient Enrichment → Template Enrichment → SMTP Delivery → Statistics
```


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

1. **Admin** leaves a note on a document paragraph via `Docs.Editor.Web` — a thin web service with no business logic, responsible only for authentication, authorization, and request validation — which issues `command.document.note.add`.
2. `Docs.Metadata.State` updates the document summary.
3. `Docs.Notifications.Workflow` kicks off a **state machine**: triggers an immediate notification, schedules a 2-hour digest reminder, and sets a 24-hour timeout — all cancellable if the user views the note earlier.
4. `Notifications.State` registers a scheduled task in DB. `Notifications.EventTicker` emits periodic tick events to check for unprocessed tasks.
5. `Notifications.Dispatcher` orchestrates enrichment — fetches recipient info from `Notifications.Account` and template data from `Notifications.Template.Store` — then sends `command.notification.email.send`.
6. `Notifications.SMTP` delivers the email and publishes `event.notification.sent` or `event.notification.failed`.
7. When the **Customer** opens the email link, `Docs.Viewer.Web` emits `event.document.note.viewed`, which cancels any pending scheduled notifications in the workflow.


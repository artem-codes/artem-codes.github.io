---
layout: page
title: "SAGA Pattern: How We Actually Implemented It"
permalink: /log/saga-pattern-real-implementation/
tags: [saga, rabbitmq, cqrs, dotnet]
---

## Context / Problem1


```mermaid
graph LR
    A:::accent0 -->|event1| B:::accent1
    B -->|event2| C:::accent2
    C --> Database[(Database)]:::accent3
```
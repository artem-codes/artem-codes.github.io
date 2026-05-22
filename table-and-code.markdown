---
layout: page
title: Experience
permalink: /experience/
---

## Experience Overview

| Company | Role | Period |
|---------|------|--------|
| Andersen | Senior .NET Engineer | Jan 2024 – Present |
| Social Discovery Group | Dev Team Lead / Senior .NET Developer | Feb 2019 – Sept 2023 |
| Scand | Senior Fullstack .NET Developer | Feb 2017 – Feb 2019 |
| Mrsoft | Fullstack .NET Developer | Oct 2014 – Feb 2017 |
| Itransition | .NET Developer | Dec 2012 – Jun 2014 |
| BiExpert | Junior .NET Developer | Aug 2011 – Dec 2012 |

---

## Code Sample

Here is an example of a simple event-driven message handler in C#:

```csharp
public class OrderCreatedHandler : IMessageHandler<OrderCreatedEvent>
{
    private readonly IOrderRepository _repository;
    private readonly ILogger<OrderCreatedHandler> _logger;

    public OrderCreatedHandler(
        IOrderRepository repository,
        ILogger<OrderCreatedHandler> logger)
    {
        _repository = repository;
        _logger = logger;
    }

    public async Task HandleAsync(OrderCreatedEvent message, CancellationToken ct)
    {
        _logger.LogInformation("Processing order {OrderId}", message.OrderId);

        var order = new Order(message.OrderId, message.CustomerId, message.Items);
        await _repository.SaveAsync(order, ct);

        _logger.LogInformation("Order {OrderId} saved successfully", message.OrderId);
    }
}
```

---

## Tech Stack

```yaml
languages: [C#, TypeScript, SQL]
frameworks: [.NET Core, ASP.NET, Angular]
messaging: [Kafka, RabbitMQ, SQS, SNS]
databases: [PostgreSQL, SQL Server, MongoDB, Redis, DynamoDB]
infrastructure: [Kubernetes, Docker, AWS EKS, Terraform]
observability: [Grafana, Datadog, Kibana, Elasticsearch]
practices: [DDD, CQRS, Event Sourcing, SAGA, BFF]
```

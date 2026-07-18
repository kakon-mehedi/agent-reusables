---
doc_type: standard
service: null
status: accepted
layer: coding
applies_to_agents: [coding-agent, api-design-agent, event-design-agent, database-design-agent, code-review-agent]
---

# Naming Conventions

| Element | Convention | Example |
|---|---|---|
| Service repo | `<org>-<noun>-service` | `acme-billing-service` |
| Domain event | `<Entity><PastTenseVerb>` | `UserRegistered`, `PaymentFailed` |
| RabbitMQ routing key | `service.entity.action` | `user.registered`, `payment.completed` |
| Kafka topic | `<org>.<service>.<entity>` | `acme.analytics.user-events` |
| REST resource path | plural noun, no verbs | `/users/{id}`, not `/getUser` |
| Database table | snake_case, plural | `order_items` |
| C# class/type | PascalCase | `UserAggregate` |
| C# variable/param | camelCase | `userId` |
| Environment variable | SCREAMING_SNAKE_CASE | `PAYMENT_GATEWAY_API_KEY` |
| Feature flag | kebab-case | `new-checkout-flow-v2` |

Rule of thumb: a name should be guessable by someone who has read the ubiquitous language glossary and nothing else.

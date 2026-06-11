# Spec Changelog

All notable changes to the **EventPeople specification** are documented here.  
This file tracks the evolution of the interface contract, not individual implementation releases.

Versioning follows [Semantic Versioning](https://semver.org):
- **MAJOR** — breaking changes to the public API (renamed methods, removed attributes, changed signatures)
- **MINOR** — new backward-compatible additions (new components, new optional attributes/methods)
- **PATCH** — clarifications, corrections, and non-breaking adjustments to existing definitions

When a MAJOR or MINOR version is released, all implementations should update their
`.event_people.yml` `spec_version` field and address any new requirements.

---

## [Unreleased]

---

## [2.0.0] — 2026-06-11

### Breaking — Queue naming convention corrected

**Queue name changes from `{appName}-{resource}.{origin}.{action}.all` to `{appName}-{resource}.{origin}.{action}.{appName}`.**

The destination segment in the queue name now identifies the consuming service (its `appName`) rather than the broadcast placeholder `.all`. This correctly models queue ownership and allows multiple services to each have their own queue for the same event type.

Migration: existing queues ending in `.all` must be deleted and recreated before deploying updated consumers. Drain the queue before deletion to avoid losing messages.

#### Updated routing convention
- Queue: `{appName}-{resource}.{origin}.{action}.{appName}` (destination = consuming app)
- Binding 1: routing key `resource.origin.action.{appName}` (targeted messages)
- Binding 2: routing key `resource.origin.action.all` (broadcast messages)
- `queueName()` must normalize the destination segment to `{appName}` regardless of input routing key (`.all` or `{appName}` both resolve to the same queue)

Includes all additions from the abandoned [1.1.0] release (retry/DLQ API surface, new env vars, RabbitMQ topology).

---

## [1.1.0] — 2026-06-11

### Added — Retry and Dead Letter Queue (DLQ)

#### API surface
- `Event.retryCount` (integer, auto-initialized to 0)
- `Event.incrementRetryCount()` method
- `Config.maxAttempts` (integer, default 3)
- `Config.delayStrategy` (string, default `"exponential"`, options: `"fixed"`, `"exponential"`)
- `Config.dlqName` (string, defaults to `{appName}_dlq` if not set)
- `Config.getRetryConfig()` method
- `Listener.on` extended: `on(eventName, callback, maxAttempts, delayStrategy, dlqName)`
- `Context.maxRetries` (integer) and `Context.isLastRetry` (boolean)
- `RabbitContext.maxRetries`, `RabbitContext.isLastRetry`, `RabbitContext.dlqName`
- `RetryManager` internal component (not exposed to users)
- New env vars: `RABBIT_EVENT_PEOPLE_MAX_RETRIES` (default 3), `RABBIT_EVENT_PEOPLE_RETRY_TTL_MS` (default 1000)

#### RabbitMQ topology (created automatically on subscribe)
- **Retry queue** (`{queue_name}_retry`): per-queue, durable. Declared without queue-level TTL; each message carries its own `expiration` property so fixed and exponential delays are both supported. Dead-letters back to the main queue via the default exchange.
- **Dead-letter exchange** (`{appName}_dlx`): per-app fanout, durable. All main queues declare `x-dead-letter-exchange` pointing here.
- **Dead Letter Queue** (`{appName}_dlq`): per-app, durable, bound to the DLX. RabbitMQ automatically adds `x-death` headers (origin queue, reason, timestamp) to every dead-lettered message — no origin information is lost.

#### Retry flow
- `fail()` with retries remaining → calculate delay (fixed or exponential) → publish to `{queue_name}_retry` with `expiration` = delay ms → ack current delivery
- `fail()` with retries exhausted → `nack(requeue=false)` → native DLX routes to DLQ
- `reject()` → `nack(requeue=false)` → native DLX routes to DLQ

#### Delay defaults
- **Exponential**: `initialDelay * (5 ^ retryCount)`, capped at `maxDelay`; defaults: initialDelay=1000ms, maxDelay=600000ms
- **Fixed**: constant delay = `RABBIT_EVENT_PEOPLE_RETRY_TTL_MS` (default 1000ms)

---

## [1.0.0] — 2026-06-11

Initial specification.

### Added
- **Event naming convention**: `resource.origin.action.destination` (destination defaults to `all`)
- **Routing convention**: queue name = `{appName}-{resource}.{origin}.{action}.all`; routing key = `{resource}.{origin}.{action}.{destination}` (no app name prefix in routing key)
- **Config** static class: `appName`, `topicName`, `vhostName`, `url`, `fullUrl`, `getBroker()`
- **Event** class: `name`, `headers`, `body`, `schemaVersion`; methods `payload()`, `hasBody()`, `hasName()`, `buildPayload()`, `generateHeaders()`, `fixName()`
- **Listener** static class: `on(eventName, callback)`
- **Emitter** static class: `trigger(events)`
- **Daemon** static class: `start()`, `stop()`, `bindSignals()`
- **Context** interface: `success()`, `fail()`, `reject()`
- **BaseListener** class: `callback()`, `bindEvent()`, `success()`, `fail()`, `reject()`, `fixedEventName()`
- **ListenersManager** static class: `bindAllListeners()`, `addListener()`
- **BaseBroker** interface: `getConsumers()`, `getConnection()`, `consume()`, `produce()`, `closeConnection()`
- **RabbitBroker** implementing `BaseBroker`
- **RabbitContext** implementing `Context`
- **Topic**: `getTopic()`, `getChannel()`, `produce()`
- **Queue**: `subscribe()`, `queueName()`
- Headers JSON keys are **camelCase**: `appName`, `resource`, `origin`, `action`, `destination`, `schemaVersion`
- Wildcard support: `*` (one word) and `#` (zero or more words)

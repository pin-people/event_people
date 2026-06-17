# EventPeople — Compatibility & Migration

This document records how the dead-letter / retry contract has changed across spec
versions, which library versions implement each behavior, and how to migrate
**without a destructive queue recreation**.

## Dead-letter contract by spec version

| Spec | Main-queue declare | DLQ routing | Drop-in upgrade? |
|---|---|---|---|
| v1.1.0 | declares `x-dead-letter-exchange = <app>_dlx` (broker DLX) | broker dead-letters `reject`/exhausted messages to a `<app>_dlx` fanout → `<app>_dlq` | **No** — the immutable arg breaks redeclare over older queues |
| **v1.1.1** (current) | **no** dead-letter argument | library **publishes** failed messages to a plain `<app>_dlq` (default exchange) and acks | **Yes** — main queue is argument-free, matching legacy declares |

> v1.1.0 was never the intended design: spec §C always called for application-level
> routing to a plain DLQ. The broker-DLX implementation deviated from §C and is the
> root cause of the upgrade crash described below. v1.1.1 realigns the implementations
> with §C and makes the rule normative.

## Library implementations

| Library | Latest pre-v1.1.1 | Behavior before | v1.1.1 status |
|---|---|---|---|
| `event_people_go` | v0.3.1 | broker DLX arg on main queue (spec v1.1.0) | **app-level DLQ** (this change) |
| `event_people_ruby` (gem `event_people`) | 1.0.7 | no DLQ/retry (pre-DLQ) | sub-card — implement app-level DLQ |
| `event_people_node` | — | no DLQ/retry (pre-DLQ) | sub-card — implement app-level DLQ |
| `event_people_python` | — | no DLQ/retry (pre-DLQ) | sub-card — implement app-level DLQ |

## Migration

### From an argument-free version (the common case) — drop-in

Upgrading from any version whose **main queue had no dead-letter argument**
(`event_people_go` ≤ v0.1.x; the ruby/node/python pre-DLQ libraries) to a v1.1.1
library is **drop-in**: the new declare is also argument-free, so the redeclare is
equivalent — **no `PRECONDITION_FAILED`, no queue recreation, in any environment.**
The library declares the plain `<app>_dlq` and the `<queue>_retry` queue idempotently
on first subscribe.

### From an argument-bearing v1.1.0 library — one-time cleanup

If an environment is already running a v1.1.0 library (`event_people_go`
v0.2.0–v0.3.1), its **main queues carry the `x-dead-letter-exchange` argument**.
Declaring them argument-free (v1.1.1) is inequivalent and will raise
`PRECONDITION_FAILED` on redeclare. This requires a **one-time** cleanup per
environment:

1. Stop the consumer.
2. Delete the argument-bearing main queues (drain or accept the in-flight loss, as
   appropriate for the environment). The `<app>_dlx` fanout exchange and the
   broker-bound `<app>_dlq` binding may also be removed; the plain `<app>_dlq`
   **queue** can be kept.
3. Start the v1.1.1 consumer — it redeclares the main queues argument-free.

After this one-time step the environment is on the argument-free contract and **all
future upgrades are drop-in**. Most environments never reached v1.1.0 in a stable
state (the v1.1.0 rollout crash-looped), so they fall under the drop-in case above.

### No broker policy, no infra prerequisite

v1.1.1 dead-letters entirely from the application (publish to `<app>_dlq`). It does
**not** require a RabbitMQ policy, an AmazonMQ configuration change, or any
broker-side dead-letter topology. There is no deploy-ordering hazard between infra
and the library.

## Reference

- Upgrade crash incident: `event_people_go` v0.1.4 → v0.3.1 on Sophia QA3 (2026-06-15),
  `PRECONDITION_FAILED - inequivalent arg 'x-dead-letter-exchange'`. Shortcut sc-22821.
- Original DLQ + bounded-retry work: sc-22500.

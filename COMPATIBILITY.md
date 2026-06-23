# Compatibility Matrix

Current spec version: **1.2.0** (see `SPEC_VERSION`)

Legend: ✅ Implemented · ⚠️ Has deviations or minor issues · 🔴 Has critical bugs · 🔵 Pending in spec · ❌ Not implemented

| Component | Ruby `1.2.1` | Python `0.1.3` | Go `v0.4.1` | Node `0.1.1` |
|-----------|:---:|:---:|:---:|:---:|
| Config | ✅ | ✅ | ✅ | ✅ |
| Event | ✅ | ✅ | ✅ | ✅ |
| Listener | ✅ | ✅ | ✅ | ✅ |
| Emitter | ✅ | ✅ | ✅ | ✅ |
| Daemon | ✅ | ✅ | ✅ | ✅ |
| Context (interface) | ⚠️ | ⚠️ | ✅ | ⚠️ |
| BaseListener | ✅ | ✅ | ✅ | ✅ |
| ListenersManager | ✅ | ✅ | ✅ | ✅ |
| BaseBroker | ✅ | ✅ | ✅ | ✅ |
| RabbitBroker | ✅ | ✅ | ✅ | ✅ |
| RabbitContext | ⚠️ | ⚠️ | ✅ | ⚠️ |
| Topic | ✅ | ✅ | ✅ | ✅ |
| Queue | ⚠️ | ⚠️ | ✅ | ⚠️ |
| RetryManager | ✅ | ✅ | ✅ | ✅ |

### Summary of open issues

Spec **v1.2.0** changes the DLQ topology (§C): the main queue is declared with **no**
`x-dead-letter-exchange` argument, and dead-lettering is performed **application-level** (the library
publishes failed messages to a plain `<app>_dlq` queue and acks the original delivery). This removes the
`PRECONDITION_FAILED` crash-loop that the v1.1.x native-DLX design caused on upgrades over queues created
by an argument-free version.

- **Go `v0.4.1`** ✅ — implements app-level DLQ (no `x-dead-letter-exchange` on the main queue; `reject()`
  and retry-exhaustion publish to `<app>_dlq` then ack, with a `nack(requeue=false)` fallback). Conforms
  to spec v1.2.0.
- **Ruby `1.2.1`, Python `0.1.3`, Node `0.1.1`** ⚠️ — still implement the v1.1.x native-DLX topology
  (declare `x-dead-letter-exchange` / bind the DLQ to a DLX fanout). They remain functional on fresh
  environments but reproduce the upgrade crash-loop and now **deviate** from spec v1.2.0 (`Queue`,
  `RabbitContext`, `Context interface`). Migration to app-level DLQ is tracked as follow-up work.

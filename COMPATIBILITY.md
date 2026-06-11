# Compatibility Matrix

Current spec version: **1.1.0** (see `SPEC_VERSION`)

Legend: ✅ Implemented · ⚠️ Has deviations or minor issues · 🔴 Has critical bugs · 🔵 Pending in spec · ❌ Not implemented

| Component | Ruby `1.0.7` | Python `0.1.1` | Go `v0.2.0` | Node `0.0.5` |
|-----------|:---:|:---:|:---:|:---:|
| Config | ✅ | ✅ | ✅ | ✅ |
| Event | ✅ | ✅ | ✅ | ✅ |
| Listener | ✅ | ✅ | ✅ | ✅ |
| Emitter | ✅ | ✅ | ✅ | ✅ |
| Daemon | ✅ | ✅ | ✅ | ✅ |
| Context (interface) | ✅ | ✅ | ✅ | ✅ |
| BaseListener | ✅ | ✅ | ✅ | ✅ |
| ListenersManager | ✅ | ✅ | ✅ | ✅ |
| BaseBroker | ✅ | ✅ | ✅ | ✅ |
| RabbitBroker | ✅ | ✅ | ✅ | ✅ |
| RabbitContext | ⚠️ | ✅ | ✅ | ✅ |
| Topic | ✅ | ✅ | ✅ | ✅ |
| Queue | ✅ | ✅ | ⚠️ | ✅ |
| RetryManager | ❌ | ❌ | ❌ | ❌ |

### Summary of open issues

See each implementation's `.event_people.yml` for the full detail on deviations and bugs.

#### Ruby
- `RabbitContext` / `BaseListener`: methods named `success!`, `fail!`, `reject!` instead of `success()`, `fail()`, `reject()` (DEV-RB-001)

#### Python `0.1.1`
No open deviations or bugs. All behavior matches spec.

#### Go `v0.2.0`
- `Queue`: `queueNameByRoutingKey()` does not normalize destination to `all` for 4-part routing keys, creating two physical queues instead of one queue with two bindings (DEV-GO-002)

#### Node.js `0.0.4`
No open deviations or bugs. All behavior matches spec.

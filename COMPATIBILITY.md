# Compatibility Matrix

Current spec version: **1.0.0** (see `SPEC_VERSION`)

Legend: ✅ Implemented · ⚠️ Has deviations or minor issues · 🔴 Has critical bugs · 🔵 Pending in spec · ❌ Not implemented

| Component | Ruby `1.0.7` | Python `0.1.0` | Go `v0.1.0` | Node `0.0.4` |
|-----------|:---:|:---:|:---:|:---:|
| Config | ✅ | ✅ | ✅ | ✅ |
| Event | ✅ | ✅ | ✅ | ✅ |
| Listener | ✅ | ✅ | ✅ | ✅ |
| Emitter | ✅ | ✅ | ✅ | ✅ |
| Daemon | ✅ | ✅ | ⚠️ | ✅ |
| Context (interface) | ✅ | ✅ | ✅ | ✅ |
| BaseListener | ✅ | ✅ | ✅ | ⚠️ |
| ListenersManager | ✅ | ✅ | ✅ | ✅ |
| BaseBroker | ✅ | ✅ | ✅ | ✅ |
| RabbitBroker | ✅ | ✅ | ✅ | ✅ |
| RabbitContext | ⚠️ | ⚠️ | ✅ | ✅ |
| Topic | ✅ | ✅ | ✅ | ✅ |
| Queue | ✅ | ✅ | ✅ | ✅ |
| RetryManager | 🔵 | 🔵 | 🔵 | 🔵 |

### Summary of open issues

See each implementation's `.event_people.yml` for the full detail on deviations and bugs.

#### Ruby
- `RabbitContext` / `BaseListener`: methods named `success!`, `fail!`, `reject!` instead of `success()`, `fail()`, `reject()` (DEV-RB-001)

#### Python `0.1.0`
- `RabbitContext`: class is named `Context` instead of `RabbitContext` (DEV-PY-001)

#### Go `v0.1.0`
- `Daemon`: no signal handling (`bindSignals` not implemented) (DEV-GO-001)

#### Node.js `0.0.4`
- `BaseListener.bindEvent()`: registers a second subscription with `.{appName}` destination in addition to `.all` — deviation from spec, but non-breaking extension (DEV-ND-001)
- `Event.fixedEventName()`: method is named differently from Ruby/Go `fixName()` — no functional impact (DEV-ND-002)

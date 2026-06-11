# Compatibility Matrix

Current spec version: **1.1.0** (see `SPEC_VERSION`)

Legend: ✅ Implemented · ⚠️ Has deviations or minor issues · 🔴 Has critical bugs · 🔵 Pending in spec · ❌ Not implemented

| Component | Ruby `1.1.0` | Python `0.1.1` | Go `v0.2.1` | Node `0.0.5` |
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
| RabbitContext | ✅ | ✅ | ✅ | ✅ |
| Topic | ✅ | ✅ | ✅ | ✅ |
| Queue | ✅ | ✅ | ✅ | ✅ |
| RetryManager | ❌ | ❌ | ❌ | ❌ |

### Summary of open issues

See each implementation's `.event_people.yml` for the full detail on deviations and bugs.

All implementations are fully compliant with spec v1.1.0 for the components listed above.
The only open item across all implementations is `RetryManager` (❌ not yet implemented).

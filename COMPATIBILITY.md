# Compatibility Matrix

Current spec version: **1.2.0** (see `SPEC_VERSION`)

Legend: ✅ Implemented · ⚠️ Has deviations or minor issues · 🔴 Has critical bugs · 🔵 Pending in spec · ❌ Not implemented

| Component | Ruby `1.2.1` | Python `0.2.1` | Go `v0.4.0` | Node `0.2.0` |
|-----------|:---:|:---:|:---:|:---:|
| Config | ✅ | ✅ | ✅ | ✅ |
| Event | ✅ | ✅ | ✅ | ✅ |
| Listener | ✅ | ✅ | ✅ | ✅ |
| Emitter | ✅ | ✅ | ✅ | ✅ |
| Daemon | ✅ | ✅ | ✅ | ✅ |
| Context (interface) | ✅ | ✅ | ✅ | ✅ |
| BaseListener | ✅ | ⚠️ | ✅ | ✅ |
| ListenersManager | ✅ | ✅ | ✅ | ✅ |
| BaseBroker | ✅ | ✅ | ✅ | ✅ |
| RabbitBroker | ✅ | ✅ | ✅ | ✅ |
| RabbitContext | ✅ | ✅ | ✅ | ✅ |
| Topic | ✅ | ✅ | ✅ | ✅ |
| Queue | ✅ | ✅ | ✅ | ✅ |
| RetryManager | ✅ | ✅ | ✅ | ✅ |

### Versions

| Implementation | Version | Declares spec | Published |
|----------------|---------|:---:|-----------|
| Ruby | `1.2.1` | 1.2.0 | RubyGems `1.2.1` |
| Python | `0.2.1` | 1.2.0 | PyPI `0.2.1` |
| Go | `v0.4.0` | 1.2.0 | git tag `v0.4.0` |
| Node | `0.2.0` | 1.2.0 | not published yet: npm is at `0.1.1`, which predates spec v1.2.0 |

"Declares spec" is the `spec_version` field of each implementation's `.event_people.yml`.

### Summary of open issues

The component status above is the one each implementation declares for spec v1.2.0. This revision
re-read the Python implementation only; Ruby, Go and Node were not re-audited.

- **Python — BaseListener (⚠️):** `bind_event(event_name, callback)` takes its arguments in the
  opposite order of the contract's `bindEvent(function, eventName)`.
- **Python — fixed in `0.2.1`:** `Event` read inbound payloads as a Python literal instead of JSON,
  and `Daemon.start()` returned without consuming. Both affect `0.1.3` and `0.2.0`; do not use
  those versions for consumers.
- **Node:** the version that implements spec v1.2.0 (`0.2.0`) is not on npm yet.

# Operations Runbook — Sessions

*Last revised 2026-07-10 after the June incident.*

## Where sessions live

Redis, single primary with one replica. Database 0. Keys are prefixed `sess:`.

## Configuration in production

| Setting | Value | Notes |
|---|---|---|
| `SESSION_TTL` | `86400` | Seconds. **24 hours.** Raised from 1800 on 2026-06-02. |
| `SESSION_IDLE` | `900` | Seconds. 15 minutes. Unchanged. |
| `REDIS_MAXMEMORY_POLICY` | `noeviction` | Sessions must not be evicted under pressure. |

The TTL was raised after the June incident, when the 30-minute expiry logged users out mid-checkout
during a slow third-party payment callback. Support volume dropped immediately afterwards.

## Restart procedure

Draining a Redis primary destroys all live sessions. Every user is logged out. Schedule outside
business hours.

## Known gaps

Password change does not currently invalidate existing sessions. A user who changes their password
because they suspect compromise stays logged in on the attacker's device until the TTL expires.
Tracked, not scheduled.

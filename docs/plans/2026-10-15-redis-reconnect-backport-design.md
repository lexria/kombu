# Stable Redis reconnect backport

Date: 2026-10-15. Tier A: this dependency changes broker recovery behavior.

The user approved a targeted stable backport of five upstream Redis fixes,
not a beta release or a runtime monkeypatch. The consuming application's plan
already covers this dependency, lock provenance and isolated real-worker proof.
This fork-specific checkpoint makes its exact source artifact independently
reviewable before publication. It does not replace the application review gates.

Base: upstream Kombu v5.6.2, `279b81f3042f23524111d3afcc773e8bb5530672`.
Approved upstream commits (each has one parent):

| PR | Commit | Purpose |
| --- | --- | --- |
| 2492 | `6a37089db8f9fbca2c0a6401d312f8efa70e7a73` | Retain the poll callback and remove stale disconnected descriptors. |
| 2498 | `2c8372c6b35b6c61bb6cf93470c57d9eab8836b8` | Handle timer connection errors and replace stale reconnect timers. |
| 2548 | `df3693367bfc664c3bb7f2a928f888513fef6234` | Preserve health-check kwargs and support redis-py 7.x metadata. |
| 2561 | `4281680ef3a275a7d87433a703790251d9805803` | Ignore stale/unmapped descriptors safely. |
| disconnect kwargs | `d9c985fee043264d3736f52d92a9cfd31b84c272` | Forward redis-py 7.3.0 disconnect error/failure keywords instead of crashing recovery. |

The fifth fix is explicitly user-approved following independent implementation
review I01-001 and a no-network reproduction on the actual candidate wheel.
It changes two runtime lines; preserve both upstream unit/integration regressions
with the whole cherry-pick. The env-gated upstream integration owner is retained
for provenance but is not run against an ambient Redis endpoint. The consuming
application still owns its isolated prefork recovery/no-duplicate proof. No other
upstream idle/overdue BRPOP fixes are approved. That proof must demonstrate
blackhole recovery with the selected artifact and bounded transport policy before
operational use; a failure cannot be waived by these green library unit tests.

The consuming application's already-approved command policy is
`retry_on_timeout=False`, with `retry_on_error` absent. This is separate from
Celery's enabled broker/blueprint reconnect. Prove the actual native async pool
has zero command retries and no retry-on-error list. Keep that policy: do not
silently enable in-place native command retry. With native retry enabled, the
four/five-fix transport can leave stale `_chan_to_sock` registration after a
health disconnect and silently stall; upstream #2630/`29a17e48` addresses that
broader case but is not authorized here. Record this limitation and verify the
selected no-command-retry health-failure path using the real redis-py machinery.
The application must also prove a server-closed idle connection after its health
check comes due, before operational use. This constrains verification to the
existing approved policy, rather than expanding the upstream patch set.

Resolve #2548's requirements conflict to exactly
`redis>=4.5.2,!=4.5.5,!=5.0.2,<8.0.0`. Preserve the stable floor/exclusions and
change only its upper cap. Do not import the unrelated newer >=5.3.1 floor.
Verify the built wheel's `Requires-Dist` Redis extra has exactly that specifier
set and accepts redis-py 7.3.0, offline, before attesting the artifact.

The cheapest adequate option is these five changes and their upstream regressions
on the stable tag, estimated one engineer-day including review and proof. The
chosen option is the same: no extra runtime abstraction or feature. Waiting for
a later stable release leaves the known defect unresolved; a broad beta brings
unrelated changes and is not authorized. No new stronger correctness standard is
introduced here. These are effort estimates, not measured timings.

Only `kombu/transport/redis.py`, `requirements/extras/redis.txt`, the existing
Redis unit-test owner, the fifth fix's `t/integration/test_redis.py` hunk and these dated provenance documents should differ from
the stable base. Preserve upstream authorship/provenance and license. Keep the
distribution version 5.6.2; the immutable Git commit identifies the backport.
No application code, client data, credentials or production topology belongs here.

Use the upstream Redis unit owner for callback/timer/descriptor/kwargs coverage.
Retain independent installed-stack checks with redis-py 7.3.0 for plain and SSL
effective health-check parameters. Red diagnostics against unpatched 5.6.2 prove
the latent defects; green library tests do not prove the historical incident or
the real-worker reconnect requirement. The consuming application owns its isolated
Redis/prefork disconnect/blackhole/no-duplicate tests before operational use.

No public fork, push, tag, release, dependency deployment or production mutation
is authorized by this checkpoint. Prepare a local immutable artifact, obtain
independent implementation review and attestation, then request named publication
authority. The application's local process-scoped URL rewrite permits prepublication
lock/proof without modifying user-global Git config. Fresh public-URL frozen
installation must later match this exact commit; changed bytes require re-review.
Remove the override when a stable upstream release includes the reviewed fixes.

Residuals: this addresses established library failure paths, not every network
outage or a proven historical root cause. Application timeouts, recovery settings,
execution evidence and safe handling of early-ack tasks remain necessary.

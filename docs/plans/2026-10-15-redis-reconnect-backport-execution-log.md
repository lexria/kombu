# Stable Redis reconnect backport execution log

Date: 2026-10-15. Tier A: broker recovery dependency artifact.

The user approved the targeted stable backport and explicitly required the agent
review loop for plans. The parent application's reviewed dependency scope is
unchanged; this checkpoint independently reviews the actual third-party source
artifact and its public provenance before any publication. No gate is waived.

Fetched an unmodified upstream bare repository, verified v5.6.2 resolves to
`279b81f3042f23524111d3afcc773e8bb5530672`, and created the local
`redis-reconnect-backport` worktree from the fixed `kombu-v5-6-2-base` branch.
Confirmed all four named upstream commits have a single parent. Only these three
planning documents have been authored; no runtime/test patch has been applied.

Self-review: the plan restricts runtime changes to the approved upstream fixes,
uses existing Redis regression ownership and source-identity checks, keeps all
verification local, and separates local artifact approval from publication and
application/production proof. `git diff --check` is the planning check; no library
test or reconnect success is claimed before implementation.

Independent plan review: next through `cross-agent-review-loop`, task
`kombu-redis-reconnect-backport`, base `kombu-v5-6-2-base`. Independent implementation
review: not started. No public repository, push, release, deployment or external
state change is authorized or performed. Append findings/dispositions, measured
verification and exact commit identities as the checkpoint advances.

## Independent plan review 001 remediation

The independent reviewer returned `changes_requested`, with one blocker and one
advisory. Both are accepted for `fix_now` before source changes. P01-001 fixes the
Redis-extra metadata conflict precisely: preserve stable >=4.5.2 and both excluded
versions, raise only the cap to <8.0.0, and prove the actual built metadata admits
7.3.0. No unapproved intermediate minimum-version bump is imported.

P01-002 defines the expected red node set per upstream PR before execution,
expected green controls, source-path/version assertions and a neutral-directory
preload mechanism so pytest cannot silently test the candidate runtime in the
stable red comparison. Collection/import/setup errors are not acceptable defect
proof. The unchanged owner baseline, focused timing and green run stay required.
Source remains unmodified; only these three plans differ. Resubmit this concrete
metadata/proof correction through the loop before cherry-picking.

## Plan approved; stable baseline diagnosis

Independent plan review 002 approved both corrections with no new findings,
finalized automatically and attested to the exact documentation commit
`4ae1ff662f1be28341a4f8666337cb8f517fe199`. Implementation review remains required.
Provisioned a fork-owned Python 3.14 environment with stable kombu 5.6.2 (no
Redis extra), redis-py 7.3.0 and the planned focused test tools.

The initial stable-owner run had four failures: the existing health-check
parameter KeyError and three harness effects, not four established defects.
Preloading the Redis transport before the unit module installed its fake poller
bound the native epoll to fake descriptors. Also, two Sentinel cases deliberately
expect an unreachable local endpoint, while the network guard raised the wrong
exception type. No real socket connection was made.

Corrected the verification harness within the approved source-identity invariant:
preload the package hierarchy only, assert the transport path after collection,
and simulate connection-refused only for the owner's explicit loopback:65534
endpoint; every other real socket connect still fails immediately. No production
code or library behavior is patched by that harness. The implementation plan
records the concrete setup correction; it remains part of implementation review.

The corrected unchanged owner baseline is **126 passed, one expected existing
KeyError in test_connparams_health_check_interval_supported**, 35.91s, exit 1.
That existing failure is the health-kwargs defect addressed by #2548 and is added
prospectively to the candidate red comparison. No import/setup failures count as
red evidence. The native retry waits in the two upstream negative Sentinel cases
explain most runtime; each body remains capped at 60 seconds. Source/test patches
have not yet been applied. Preserve all expected node IDs before the next run.

## Candidate source and measured verification

Applied only the four approved upstream commits with `cherry-pick -x`, preserving
their authors. The only conflict was #2548's requirement line, resolved exactly
to the reviewed stable floor/exclusions and <8.0.0 upper cap. The runtime source
checkpoint is `ff454b7942bf2d64c6360290f3d3fff073baa874`; the publication candidate
also includes this verification record. No other runtime/test files changed.

The prospective expected-red manifest named 17 nodes and 127 green controls.
Candidate tests against the unpatched installed library produced **exactly 17
failures and 127 passes in 30.71s**, exit 1. Parsed JUnit matched both complete
node sets, with zero errors or skips; no unexpected failure was relabeled.
The installed stable transport SHA256 was
`27d57f1acd3dd210eb90ec26dadca7c36072814ae88fb6bfaa060a81cde66655`.

Built the candidate wheel using `uv build --wheel`, inspected its METADATA with
`email.parser` and `packaging.Requirement`, and required exactly one Redis-extra
requirement with `SpecifierSet('>=4.5.2,!=4.5.5,!=5.0.2,<8.0.0')`. It admits
7.3.0; distribution version remains 5.6.2. Installed that wheel into the separate
test environment with `uv pip install --reinstall --no-deps`, preserving redis
7.3.0. The identical test owner/harness then produced **144 passes in 38.49s**,
exit 0. Its installed transport SHA256 was
`3b6bf5d705d010a6f0c38b8c73bd7c99915e4c24f5bc2305eec7ee83afcb03ee`.
Both runs asserted imported package/transport paths inside the separate installed
environment, the expected transport hash, and versions at collection finish.
Neither run imported runtime source from the checkout.

Harness mechanics: run from a neutral directory; import installed `kombu` and
`kombu.transport` before pytest; let the owner install its fake poller before
loading `kombu.transport.redis`; assert runtime identity in
`pytest_collection_finish`. Invoke only the Redis owner with `-q --timeout=60
-o timeout_func_only=true --junitxml=<report>`. Guard `socket.connect` and
`connect_ex` against real networking. The two explicit localhost:65534 Sentinel
negative cases receive simulated ECONNREFUSED; every other attempted connection
raises AssertionError. JUnit node sets are compared against the prospective
manifest after normalizing only the absolute owner prefix. This is test setup,
not a patch to the transport or its assertions.

An additional no-network diagnostic constructed actual async `_connparams()`
for both `redis://localhost:6379/0` and `rediss://localhost:6379/0`, with
`Channel._create_client` mocked and socket connections forbidden. Both preserve
the requested health interval 30; their connection classes inherit the expected
plain/SSL redis-py class. The upstream owner supplies callback survival, timer
replacement/error handling and stale-descriptor coverage; no duplicate owner was
added. `python -m flake8 kombu/transport/redis.py t/unit/transport/test_redis.py`
through the fork-owned uv environment and `git diff --check` both pass.

Verification remains library-level. No real-worker recovery, historical-cause
certainty, fresh public fetch, application install, CI/Docker proof or production
activation is claimed. Independent implementation review and attestation follow
against the exact local candidate before any named publication request.

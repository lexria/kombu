# Stable Redis reconnect backport implementation plan

Date: 2026-10-15. Tier A. Independent plan review before source changes;
independent implementation review and exact-commit attestation before publication.

1. Verify the stable tag and five upstream commit identities in the design.
   Record the independent plan result and dispositions before applying source.
2. Provision a fork-owned Python 3.14 test environment; do not modify another
   project's virtualenv. Use pytest 8.4.2, pytest-timeout, pytest-freezer and
   redis-py 7.3.0 plus the upstream owner's actual fixture dependencies. Select
   only `t/unit/transport/test_redis.py`, not the entire library suite.
3. Apply the five single-parent upstream commits with provenance (`cherry-pick -x`),
   in the order in the design. Resolve context against stable behavior rather
   than importing unrelated newer source. Stop on a newly required runtime change
   outside the five fixes and re-review the scope. Preserve the release version.
   #2548's Redis requirements hunk conflicts with stable: resolve it to exactly
   `redis>=4.5.2,!=4.5.5,!=5.0.2,<8.0.0`, preserving the floor/exclusions and
   changing only the cap. Record that deliberate stable-context resolution.
4. Run the upstream added callback/timer/kwargs/descriptor regressions against
   the unpatched installed library to retain red evidence, explicitly asserting
   the imported source path. Then install the reviewed fork and run the complete
   Redis unit owner with redis-py 7.3.0. Never accidentally import the fork for
   the red run or a cached stable package for green.
   Install stable `kombu==5.6.2` without its Redis extra and `redis==7.3.0`
   separately. Establish the unchanged owner baseline first. For the new-test red
   comparison, run from a neutral directory and preload the installed `kombu`
   package hierarchy, asserting its source path. Leave Redis transport loading
   until the unit module installs its fake poller; assert that transport's path
   at collection finish. This prevents root conftest path insertion choosing the
   candidate runtime without breaking the owner's fake-descriptor setup. Record
   redis-py version and imported paths in each run.
   A collection/import/setup error never counts as red; require only the named
   assertion failures or targeted KeyError/ConnError below, plus only the new
   disconnect node's exact `TypeError: Channel._connparams.<locals>.Connection.disconnect()
   got an unexpected keyword argument 'error'`, with all other cases
   green. Expand parametrized IDs via collect-only and fix the expected set
   before running the red comparison.

   All node IDs below use prefix `t/unit/transport/test_redis.py::`:

   | Fix | Expected-red node suffixes on stable runtime |
   | --- | --- |
   | 2492 | `test_Channel::test_register_with_event_loop__on_disconnect__loop_cleanup[fds0]`; `test_Channel::test_register_with_event_loop__on_disconnect__removes_sock`; `...__sock_no_fileno`; `...__sock_none_prunes`; `...__sock_none_subclient` (same class/method prefix). |
   | 2498 | `test_Channel::test_register_with_event_loop_stores_trefs_on_cycle`; `...cancels_stale_trefs_on_reconnect`; `test_MultiChannelPoller::test_maybe_restore_messages_swallows_connection_error`; `...test_maybe_check_subclient_health_swallows_connection_error`. |
   | 2548 | `test_Channel::test_connparams_health_check_interval_supported` (already fails in the stable baseline with redis-py 7.3.0); `test_RedisSentinel::test_getting_master_from_sentinel` and `_single_node`, `_with_client_name`, `_with_acl_credentials`, `_with_password_only` variants. |
   | 2561 | `test_MultiChannelPoller::test_on_readable_ignores_unmapped_fd`; `test_MultiChannelPoller::test_handle_event_ignores_unmapped_fd`. |
   | disconnect kwargs | `test_Channel::test_redis_disconnect_forwards_keyword_arguments`. |

   Before applying the fifth fix, obtain independent review of this revised plan
   and the consuming application's matching dependency-scope update. Preserve
   the four-fix candidate and its failed implementation review. Retain the fifth
   commit's upstream env-gated integration hunk without running it against an
   ambient broker. Collect the expanded unit owner and prospectively extend the
   manifest to 18 expected failures and 127 green controls. Repeat stable-runtime
   red, candidate-wheel green, both URL diagnostics, source-hash assertions and
   built metadata checks. The reproduced keyword TypeError is additional targeted
   red evidence, not a replacement for that complete comparison.
   Also repeat that reproduction as green through real redis-py 7.3.0
   `Connection.check_health()`/Retry/`_ping_failed`, obtaining the actual async
   connection from the `_connparams(asynchronous=True)` native connection pool.
   Simulate only the PING failure and forbid socket connects. Use the parent's
   already-approved `retry_on_timeout=False`, absent `retry_on_error`; assert
   zero native command retries and an empty retry-on-error list. The four-fix
   wheel must reproduce the keyword TypeError without closing the simulated
   socket. The five-fix wheel must close it and surface the original
   `redis.exceptions.ConnectionError`, never TypeError. A forwarding assertion
   against a mock base class alone cannot close I01-001.
   Parent scope includes one actual server-closed idle-connection case after its
   health check comes due under this same policy. Native retry-enabled behavior
   and #2630 are explicitly outside this approved artifact, not silently healthy.

   Expected-green controls include loop_cleanup[fds1], fileno_oserror,
   fd_not_in_map, fileno_negative, the maybe_restore_messages happy/no-active paths,
   subclient-health happy/no-subclient paths and the hardened legacy initializer
   case. Other unchanged owner tests remain green. Unexpected differences stop
   the comparison for diagnosis rather than being relabeled as intended failures.
5. Verify effective health_check_interval on plain and SSL URL connection
   parameters without opening sockets. Confirm the poll callback survives
   disconnect, reconnect timer references are replaced, timer connection errors
   remain bounded and stale descriptor events are safe. Reuse the upstream owner;
   do not duplicate its full matrix. No live/shared Redis or database is used.
6. Bound each test body to 60 seconds, keep logs and exact exit statuses, and
   record selected counts, imported package/source identities and focused timing.
   Use the repository's existing static-check configuration for changed files;
   `git diff --check` must pass. Long commands run detached with log/exit files.
   Build a wheel with the existing packaging backend and inspect its METADATA
   using email/packaging parsers. Require exactly one Redis-extra requirement,
   the design's specifier set, and `Version('7.3.0') in requirement.specifier`.
   Do not infer compatibility from the manually installed test environment.
7. Inspect the entire diff against the stable tag: only the named source/tests
   and dated provenance documents. Commit the exact candidate, independently
   review its real repository and resolve useful findings. Attest the immutable
   commit. Record outcome in the consuming application's execution log as well.
8. Leave publication pending explicit authority. The proposed public HTTPS
   source is `https://github.com/lexria/kombu.git` at the attested commit, never a
   branch or invented revision. Fresh public installation, CI/Docker fetches and
   application real-worker proof remain separate required evidence. Do not claim
   those checks pass before publication/proof actually happens.

The artifact's docs contain public upstream provenance only. Keep credentials,
application source, operational host identities and production data out of the
fork. Rollback at this stage retains the stable base ref and private candidate;
there is no installed service or external state to roll back.

# Swift identity validation

This records focused local evidence for issue #2061 and PR #2436: Swift
callable identity and conservative overload candidates, including candidate
counts and multiple trailing closure labels. The scope remains one language,
one claim.

## Tested source

The A-only checkpoint contains the seven Swift implementation and regression
test paths committed in `fix(swift): match multiple trailing closure labels`.
It was tested from a fresh `git archive` snapshot, without uncommitted
infrastructure changes or Git metadata.

Public PR head S is `b0958c6cd36c68825fac55cc58934181a2251192`; it does
not contain this local A checkpoint. A's parent is local merge
`b7c86690618ed0accf137a42a987887872475fc3`. This document and the local
checkpoint code have not been pushed. PR readiness remains unestablished.

| Identity | Value |
| --- | --- |
| Commit | `761ea8ca432ae0f58a6bb77c438ccedcb1ade3a1` |
| Tree | `b1718c8b0e14eb1bd7d2a2f6da1bda024e4a1912` |
| Archive SHA-256 | `d934685cc5e521060a8ddb29dcb65e5961c10d103763375e647a9e9da6a1617e` |
| Content manifest SHA-256 | `10011442026e4c5a4900c6b24ee19dda1ea3c1eda15e8bf88b0eec35b4ef77de` |

The archive path set matched all 2,254 tracked paths, including one symlink.
The content manifest contains sorted file SHA-256 records and symlink targets.
Its before/after comparison returned **0**, with identical hashes.

## Focused result

From the archived source root, with `VALIDATION_BUILD_DIR` set to a separate
temporary build directory:

```bash
scripts/test.sh --suites extraction,callable_sig,registry,pipeline \
  BUILD_DIR="$VALIDATION_BUILD_DIR"
```

The canonical iteration entry compiled the standard test runner and executed
exactly these four suites:

| Suite | Relevant coverage |
| --- | --- |
| `extraction` | Multiple trailing closures, bounded labels, compact/spill round trips |
| `callable_sig` | Swift signature identity, defaults, identity length bounds |
| `registry` | Conservative overload candidates, closure labels, defaults and registration order |
| `pipeline` | Serial/parallel candidates and incremental signature restoration |

Environment: Linux x86_64, GCC/G++ **16.2.1 20260810**, eight build cores.
The repository's default ASan+UBSan settings and the
`sanitized=1 test_seams=1` build-config assertion remained enabled.

- Started: **2026-10-03T01:45:48Z**.
- Finished: **2026-10-03T02:12:28Z**.
- Test command exit: **0**.
- Complete runner summary: **787 passed**.
- Archived source content remained identical after execution.

## Evidence limits and earlier failures

No attributable raw Swift negative reproduction log was found for parent
baseline `5538355530bb126c3041f7fcf8ef7a82b4bb3fec`. This evidence therefore
does not establish a red/green comparison against that parent.

Earlier checks used combined working trees containing Swift and independent
infrastructure changes, based on
`b7c86690618ed0accf137a42a987887872475fc3`. Their recorded
`git diff --binary` SHA-256 scopes differ:

- GCC TSan and daemon smoke:
  `b1c2b8cb54ab868ab7f053f1936ecc48370f7cd3123cb5f5efd23858a114982c`.
- The 131-source analyzer run, before the parser patch:
  `c0ac0734e190a79031c5a430e4cd71f52e6280bdfbd47bf9631cfcb8d397c1be`.

These results apply to their respective snapshots and were not rerun on the
A-only archive:

| Earlier check | Recorded result and scope |
| --- | --- |
| GCC TSan | Canonical full TSan leg failed: exit 2; 1,220 passed, 2 failed, 8 skipped. Daemon frontend EOF fixtures hit the existing 90-second child alarm. |
| Daemon smoke | Production daemon/standalone CLI smoke failed: exit 2; standalone CLI created a daemon socket, violating the smoke test's one-shot expectation. |
| Memory analysis | Canonical 131-source analysis remained FAIL. Verified execution reported findings; earlier provisional green results with missing-tool/parser failure risks were invalid. Bounded LLVM 21/22 comparisons over seven translation units found the same 16 diagnostic lines across parent/local/combined snapshots; that diagnostic comparison does not replace the 131-source gate. |

The focused A-only result does not establish full acceptance or PR readiness.
The earlier failures remain open in their respective scopes. No broader CLI,
daemon, Cypher, MCP or Store repair is included in this Swift checkpoint.

# Swift identity validation

Scope: issue #2061 / PR #2436, Swift callable identity and conservative CALLS
candidates only. Bare stored names remain unchanged. Generic declarations,
constraints, where requirements and function-level async distinguish identities;
they do not provide Swift compiler type inference. Whitespace/comments normalize;
generic parameter renaming, reordered constraints and inline/where semantic
equivalence are not canonicalized. Existing signature/argument limits remain
fail-closed. Callable identity modes for other languages remain disabled.

Format 3 rebuilds format 1/2 indexes once because generic/async identities change
persisted QNs and old CALLS edges cannot safely be updated in place. The Swift
migration regression fabricates collided nodes and an incompatible edge, verifies
source positions and candidate counts after rebuilding, then no repeat migration.
The TS/HTML/SCSS rebuild regression remains.

## Current repair evidence

Review checkpoint: `39d5db48a83f94641fccbb44c8cbc0233b5e2a51`, upstream base
`268a9d8886642eb7f9b2ce45f5ce27cdecf0f519`. Regression-only RED checkpoint:
`7188414d0a301574ed26a95c23159f06b0749cf7`. Tested production source:
`f2da1f464b31117c10ba5c169fad023ac9b133e6`. Subsequent evidence-only commits
do not imply these commands were rerun at their SHA.

Environment: remote-dev, Linux x86_64, GCC 16.2.1, glibc 2.44,
Clang/clang-format/clang-tidy 22.1.8, Cppcheck 2.21.1, GNU Make 4.4.1.
Cppcheck and its dependency were extracted from signature-verified distribution
packages into a temporary directory; no system install or new suppression.
All times below are 2026-10-05 UTC. Build/test/lint ran remotely, not locally.

| Command/path | Time | Exit/result |
| --- | --- | --- |
| `scripts/test.sh --suites extraction,callable_sig,registry,pipeline,index_format,infrascan,edge_types_probe` | 06:41:21–07:06:59 | 0; 907 passes, two ObjectScript UBSan reports; **not sanitizer-clean** |
| `scripts/test.sh --tsan TEST_TSAN_SUITES=pipeline` | 06:41:22–06:53:14 | 0; 326 passes |
| Portable graph driver, strict ASan/UBSan/LSan | 06:51:26 | 0; 10/10 semantic observations |
| Portable real JSON-RPC MCP driver, strict ASan/UBSan/LSan | 06:51:46 | 0; 5/5 envelope/tree/count observations |
| `scripts/test.sh` | 06:41:24–06:42:55 | 1; version-metadata preflight, full C suite not reached |
| `timeout 900 scripts/lint.sh` | 06:41:26–06:56:26 | 124; clang-tidy diagnostics, cppcheck unfinished |
| `timeout 900 scripts/lint.sh --ci CPPCHECK="$cppcheck -j4"` | 06:56:26–07:08:59 | 0; complete CI lint, same rules/source scope/suppressions, parallel execution only |
| `scripts/build.sh BUILD_DIR=build/dav64-swift-prod` | 06:52:31–07:08:18 | 0; clean production build isolated from test artifacts |
| Production `--version` / `--help` | 07:08:48 | 139 / 139; native startup failure |
| DCO range, memory-core, NOLINT whitelist, whitespace | 06:42:03–06:42:12 | 0; 39 nonmerge commits signed off |

The repaired source/header/test files also pass formatter dry-run. `$cppcheck`
denotes the verified temporary executable, with its library directory in
`LD_LIBRARY_PATH`; an installed Cppcheck needs no such temporary-path setup.
Strict drivers set `ASAN_OPTIONS=detect_leaks=1:halt_on_error=1` and
`UBSAN_OPTIONS=halt_on_error=1`. Completed clang-tidy
diagnostics mapped against `git diff -U0` contain zero errors on added PR lines;
that static mapping does not turn the full lint failure into a pass.

Public [RED output and readonly graph](https://gist.github.com/DavidHLP/af1ac103a4fea0fa956ff0229ee6d1aa)
show six declarations collapsed to two nodes at lines 5/7, merged marker
outgoing edges, and caller candidate_count=2. Failed-assertion cleanup also
reports leaks; that RED run is not a sanitizer-clean result.
[Final-source raw outputs, run SHAs/times/exits and binary hashes](https://gist.github.com/DavidHLP/2218c816c21c5c2468c6c83990ff225f/acda57ce32afb31678d43a177ad2f44d116ac4c8)
are public. Large raw files remain complete despite API preview truncation;
downloaded runtime/lint hashes were checked against their originals.

[Same-condition main controls](https://gist.github.com/DavidHLP/a8dcbcc4042a898575f9cb1f330377fe/269b55ffde3bc9282155a4e078cb42dc9c982400)
at `268a9d8886642eb7f9b2ce45f5ce27cdecf0f519` reproduce the Scoop newest-release
pin preflight failure (exit1), CI lint900s timeout, and full clang-tidy exit2.
The clean main production build exits0; `--version`/`--help` both exit139 with
the same `mi_free → newlocale → libstdc++ locale` startup stack. This confirms
that specific native failure is present on main too; its root cause is not
established. No equivalent parent suite run establishes attribution of the
ObjectScript reports. Current macOS/Windows, Ubuntu production CLI semantic
checks and the full C test suite remain unverified. Upstream CI/review are
separate merge gates; these results are not a maintainer approval.

Review disposition: Confirmed identity/negative-test/migration/documentation
issues are repaired; Already Addressed literal-type narrowing stays removed;
False Positive: none established; Out of Scope: other languages, compiler type
inference and unrelated baseline fixes. Full relevant source contexts were read
after the stale graph required direct-source fallback.

Portable graph/MCP commands are in the
[reproduction README](../tests/repro/issue2061_swift_identity/README.md).
Run `scripts/test.sh --suites extraction,callable_sig,registry,pipeline,index_format`
for regressions, `scripts/test.sh` for the full gate, `scripts/lint.sh` for lint,
and `scripts/test.sh --tsan TEST_TSAN_SUITES=pipeline` for parallel paths.
Standalone graph/MCP drivers are opt-in, not CI gates.

## Public historical archive

The [complete previous validation record](https://github.com/DavidHLP/codebase-memory-mcp/blob/39d5db48a83f94641fccbb44c8cbc0233b5e2a51/docs/SWIFT_IDENTITY_VALIDATION.md)
and [actual outputs and probes](https://github.com/DavidHLP/codebase-memory-mcp/tree/39d5db48a83f94641fccbb44c8cbc0233b5e2a51/tests/repro/issue2061_swift_identity/evidence/2026-10-05)
remain public at an immutable fork commit, with actual tested SHAs, commands,
timestamps, environments, exits and failures. They are historical records,
not current operation instructions or final-repair acceptance.

Historical `0f294df8c1fc1be3b00e948150c2f54f06708cb3` results: 825 focused
passes/exit 0 with two ObjectScript UBSan reports (not sanitizer-clean); graph
10/10 and MCP 5/5 strict sanitizer-clean; pipeline TSan 325 passes/exit 0;
broad TSan timeout 124; full test stopped at release-metadata preflight;
full lint failed, CI-mode lint timed out; native CLI startup failed, Ubuntu
production CLI passed. These separate results do not imply full acceptance.

Historical MCP run `0007c1d22858e1548ca392f382275595a0a2c691` exited 1
with a 1,024-byte leak after four observations; observation five and final summary
were incomplete. MCP/Store source equality with its parent alone does **not**
establish inherited runtime behaviour. Attribution remains unconfirmed without
a same-condition parent execution; neither inherited nor Swift-caused is proven.
Archived wording claiming inherited is superseded here.

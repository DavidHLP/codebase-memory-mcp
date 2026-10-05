# Swift identity validation

Scope: issue #2061 / PR #2436, Swift callable identity and conservative CALLS
candidates only. Bare stored names remain unchanged. Generic declarations,
constraints, where requirements and function-level async distinguish identities;
they do not provide Swift compiler type inference. Whitespace/comments normalize;
generic parameter renaming, reordered constraints and inline/where semantic
equivalence are not canonicalized. Existing signature/argument limits remain
fail-closed. Other languages remain disabled.

Format 3 rebuilds format 1/2 indexes once because generic/async identities change
persisted QNs and old CALLS edges cannot safely be updated in place. The Swift
migration regression fabricates collided nodes and an incompatible edge, verifies
source positions and candidate counts after rebuilding, then no repeat migration.
The TS/HTML/SCSS rebuild regression remains.

## Current repair evidence

Review checkpoint: `39d5db48a83f94641fccbb44c8cbc0233b5e2a51`, upstream base
`268a9d8886642eb7f9b2ce45f5ce27cdecf0f519`. Regression-only RED checkpoint:
`7188414d0a301574ed26a95c23159f06b0749cf7`. Repair validation is pending.
No historical pass is claimed for the repair. Final command/result records will
be added after execution. Build/test/lint run on remote-dev, not locally.

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

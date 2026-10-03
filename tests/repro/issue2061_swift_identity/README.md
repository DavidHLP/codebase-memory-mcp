# Original Swift issue #2061: standalone pipeline/store proof

This runner embeds only the three original Swift files from
[issue #2061](https://github.com/DeusData/codebase-memory-mcp/issues/2061).
It uses the real production pipeline in FAST mode and queries its persisted
store. It neither implements a resolver nor copies a new resolver into P.
It is deliberately absent from `ALL_TEST_SRCS`, `TEST_REPRO_SRCS` and CI.

**Review this checkpoint on GitHub before building or running either side.**
Syntax checks do not establish RED/GREEN. This is a focused graph proof,
not a full gate or a test of the `trace_path` presentation layer.

The runner prints every observation and a final summary. It checks two
`work` nodes, their original file/line ranges, flag → target, name ↛ target,
caller → name, caller ↛ flag, and absence of the caller in target's inbound
CALLS traversal through depth three. Node lookup uses names, files and lines,
so it works with both P's unsuffixed QNs and A's suffixed QNs. Missing
endpoints produce unavailable edge observations (`-1`), never successful
absence checks. All semantic checks are collected rather than failing early.

Exit meanings: **0** = all observations pass; **1** = semantic mismatches
after successful indexing/store access; **2** = setup, pipeline or store API
failure. A compiler/linker failure, timeout, signal, or sanitizer diagnostic
is an infrastructure/runtime failure, not the target RED. Only an exit-1
run with the expected collapse/false-caller observations and no runtime
diagnostics establishes the baseline reproduction.

## Syntax only (before review)

From each exact source archive, using the same external checkpoint files:

```bash
make -f "$harness/syntax.mk" issue2061-syntax \
  CC=gcc CXX=g++ TEST_SEAMS=1 BUILD_DIR="$build_dir" \
  ALL_TEST_SRCS="$harness/main.c"
```

`syntax.mk` only includes the archive's unmodified `Makefile.cbm` and adds
one `-fsyntax-only` recipe using its standard `CFLAGS_TEST`. It builds no
objects, links no executable and runs no fixture. It does not change flags.

## Comparable P/A build and run (after root review)

Run in Bash from the repository root. Set `reviewed_checkpoint` to the full
SHA root actually reviewed. Use one checkpoint's harness for both sides.
All archives, binaries, logs and generated fixtures stay outside Git.
No worktree or tracked source modification is required.

```bash
set -euo pipefail
reviewed_checkpoint=REPLACE_WITH_ROOT_REVIEWED_FULL_SHA
evidence=$(mktemp -d /tmp/dav64-issue2061-XXXXXX)
mkdir "$evidence/harness"
git archive "$reviewed_checkpoint" tests/repro/issue2061_swift_identity \
  | tar -x -C "$evidence/harness"
harness="$evidence/harness/tests/repro/issue2061_swift_identity"
P=5538355530bb126c3041f7fcf8ef7a82b4bb3fec
A=cf67f49dc2d718709846adefff5bab6cf9b671d4
gcc --version > "$evidence/toolchain.txt"
g++ --version >> "$evidence/toolchain.txt"
uname -a > "$evidence/platform.txt"
sha256sum "$harness/main.c" "$harness/syntax.mk" > "$evidence/harness.sha256"

for side in P A; do
  sha=${!side}
  mkdir "$evidence/$side-source"
  git archive "$sha" > "$evidence/$side.tar"
  tar -xf "$evidence/$side.tar" -C "$evidence/$side-source"
  printf '%s\n' "$sha" > "$evidence/$side.commit"
  sha256sum "$evidence/$side.tar" > "$evidence/$side.archive.sha256"
  build_dir="$evidence/$side-build"
  (
    cd "$evidence/$side-source"
    make -n -f Makefile.cbm "$build_dir/test-runner" \
      CC=gcc CXX=g++ TEST_SEAMS=1 BUILD_DIR="$build_dir" \
      ALL_TEST_SRCS="$harness/main.c"
  ) > "$evidence/$side.build-plan.txt" 2>&1
done
```

Before compiling, inspect both plans: the only test source must be the
external `main.c`; compiler, effective flags and dependency source sets must
be comparable (normalize the P/A source/build paths). Stop and report any
unexplained difference. The standard target name `test-runner` is retained,
but `ALL_TEST_SRCS` replaces the entire test source list with this one main.
The archive's standard production/extraction/grammar dependencies and
sanitizer flags are retained. No all-tests runner or suite is compiled/run.

Then build **only the binary target**, never `test`, `test-focused` or
`test-repro`. For each side, record UTC start/end and the actual build exit:

```bash
for side in P A; do
  build_dir="$evidence/$side-build"
  date -u +%FT%TZ > "$evidence/$side.build-start"
  set +e
  (
    cd "$evidence/$side-source"
    make -j8 -f Makefile.cbm "$build_dir/test-runner" \
      CC=gcc CXX=g++ TEST_SEAMS=1 BUILD_DIR="$build_dir" \
      ALL_TEST_SRCS="$harness/main.c"
  ) > "$evidence/$side.build.stdout" 2> "$evidence/$side.build.stderr"
  rc=$?
  set -e
  printf '%s\n' "$rc" > "$evidence/$side.build.exit"
  date -u +%FT%TZ > "$evidence/$side.build-end"
  if [ "$rc" -ne 0 ]; then
    printf 'BUILD FAILURE on %s; not semantic RED\n' "$side"
    exit "$rc"
  fi
  sha256sum "$build_dir/test-runner" > "$evidence/$side.binary.sha256"
done
```

Use the same inherited environment for both runs. Record an allowlist of
relevant variables (compiler paths, sanitizer options, `CBM_*` resource/index
settings and locale), not a raw environment dump that might contain secrets.
Leave resolver controls enabled; do not disable LSP or alter graph behavior
to force a desired result. Use fresh per-side repositories/databases and
the same working directory. The run directory must not already exist.

```bash
for side in P A; do
  date -u +%FT%TZ > "$evidence/$side.run-start"
  set +e
  (
    cd "$evidence"
    ASAN_OPTIONS=detect_leaks=1:halt_on_error=1 \
    UBSAN_OPTIONS=halt_on_error=1:print_stacktrace=1 \
      timeout 180s "$evidence/$side-build/test-runner" "$evidence/$side-run"
  ) > "$evidence/$side.run.stdout" 2> "$evidence/$side.run.stderr"
  rc=$?
  set -e
  printf '%s\n' "$rc" > "$evidence/$side.run.exit"
  date -u +%FT%TZ > "$evidence/$side.run-end"
done
sha256sum "$harness/main.c" "$harness/syntax.mk" > "$evidence/harness-after.sha256"
cmp "$evidence/harness.sha256" "$evidence/harness-after.sha256"
for side in P A; do
  sha256sum "$evidence/$side-run/repo/Sources/"*.swift \
    > "$evidence/$side.fixture.sha256"
done
```

Compare fixture bytes between sides and with the issue, and verify archived
source manifests before/after execution. Retain raw stdout/stderr, actual
exits, timestamps, expanded build plans, toolchain/platform and fingerprints.
Inspect all observations and sanitizer output before attributing RED/GREEN.
The three-file graph has no dependency on a daemon or a new CLI architecture.
No P/A execution result is claimed by this checkpoint.

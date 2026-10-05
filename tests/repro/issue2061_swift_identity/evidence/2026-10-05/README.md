# Current delivery observations

Tested production/test source: `0f294df8c1fc1be3b00e948150c2f54f06708cb3`.
These are actual remote-dev outputs, not expected-output fixtures. See
[the validation record](../../../../../docs/SWIFT_IDENTITY_VALIDATION.md#current-main-integration--2026-10-05)
for commands, environments, exit codes, baseline attribution and limitations.

- `final-graph.stdout`: ten original reproduction observations, exit 0.
- `final-mcp.stdout`: all five JSON-RPC requests/responses and assertions, exit 0.
- `noble-cli-*.json`: five separate production CLI envelopes, validator passes.
- `objectscript-*.stderr`: strict UBSan probes against main `268a9d88`, each exit 1.

The `.txt` source records are the exact external launchers/probes used on the
remote host. They are excluded from automatic suite discovery. To reproduce,
copy them to the original external evidence paths (dropping `.txt`) and adjust
the absolute checkout paths if needed. The combined launcher includes the two
existing production-backed drivers unchanged, using separate process runs.
The CLI validator examines the saved envelopes rather than bypassing the CLI.
The ObjectScript probe compiles only main's two unchanged scanners, with
`-std=c11 -D_DEFAULT_SOURCE -g -O1 -fsanitize=address,undefined
-fno-omit-frame-pointer` and their grammar include directories. It establishes
the specific baseline defect, not full main test acceptance.

Current graph/MCP runs use strict ASan/UBSan and leak detection and report none.
This does not erase the historical MCP failure or the two unrelated UBSan
reports in the aggregate focused test run.

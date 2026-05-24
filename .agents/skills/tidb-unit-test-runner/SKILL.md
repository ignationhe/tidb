# tidb-unit-test-runner

Runs Go unit tests for a specified package or set of packages in the TiDB repository, collecting results and surfacing failures with context.

## Trigger

Use this skill when:
- A PR modifies Go source files and unit test coverage should be verified
- A developer asks to run unit tests for a specific package or directory
- CI needs to validate that existing unit tests still pass after a change
- You need to identify which unit tests are failing and why

## Inputs

| Name | Required | Description |
|------|----------|-------------|
| `packages` | Yes | Space-separated list of Go package paths (e.g. `./executor/...` or `./planner/core`) |
| `run_pattern` | No | `-run` regexp pattern to filter specific test functions (e.g. `TestHashJoin`) |
| `timeout` | No | Per-package test timeout (default: `300s`) |
| `count` | No | Number of times to run each test (default: `1`; use `2` to detect flaky tests) |
| `race` | No | Set to `true` to enable the Go race detector (default: `false`) |
| `tags` | No | Additional Go build tags (e.g. `intest`) |

## Steps

1. **Resolve packages** — Expand any `./...` globs and validate that each package exists under the repo root.
2. **Build dependencies** — Run `go build` on the target packages to catch compilation errors before testing.
3. **Execute tests** — Invoke `go test` with the supplied options, capturing stdout/stderr per package.
4. **Parse results** — Parse `go test -json` output to extract per-test pass/fail/skip status and elapsed time.
5. **Summarise** — Emit a structured summary: total passed, failed, skipped, and wall-clock duration.
6. **Report failures** — For each failing test, include the full failure message and the relevant source location.

## Output

```
Unit test results for: ./executor/...

PASS  TestHashJoinExec          (1.23s)
PASS  TestMergeJoinExec         (0.87s)
FAIL  TestIndexLookupJoinExec   (2.10s)
  --- FAIL: TestIndexLookupJoinExec (2.10s)
      join_test.go:142: expected 3 rows, got 2
SKIP  TestSlowQueryParsing      (build tag missing)

Summary: 2 passed, 1 failed, 1 skipped  (total 4.20s)
```

## Notes

- Always run with `-count=1` to disable the test result cache unless explicitly testing for flakiness.
- If `race` is enabled, expect a 2–5× slowdown; restrict to small package sets.
- Packages under `br/`, `dumpling/`, and `lightning/` use separate Go modules — prefix commands with the appropriate `cd` or use `-C`.
- Failpoint-dependent tests require failpoints to be activated first; use the **tidb-failpoint-test-runner** skill instead.

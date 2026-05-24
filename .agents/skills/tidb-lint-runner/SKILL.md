# tidb-lint-runner

Runs linting checks on Go source files in the TiDB repository, including `golangci-lint`, `gofmt`, and custom TiDB-specific lint rules.

## Trigger

Invoke this skill when:
- A pull request modifies `.go` files
- A user asks to lint or check code style
- Pre-merge validation is needed for Go source changes

## Steps

1. **Identify changed files**
   - Use `git diff --name-only origin/master...HEAD` to get the list of changed `.go` files
   - Filter out vendor, generated, and test-only files if a focused lint is requested

2. **Run gofmt check**
   ```bash
   gofmt -l $(git diff --name-only origin/master...HEAD | grep '\.go$')
   ```
   - If output is non-empty, report which files need formatting

3. **Run golangci-lint**
   ```bash
   golangci-lint run --config .golangci.yml ./...
   ```
   - Use the project's `.golangci.yml` configuration if present
   - Capture stdout and stderr for reporting

4. **Run TiDB-specific checks**
   - Check for forbidden imports (e.g., `log` instead of `github.com/pingcap/log`)
   - Verify error wrapping conventions using `errors.Trace` / `errors.Annotate`
   - Ensure context propagation is not dropped in new functions

5. **Report results**
   - Summarize pass/fail per check category
   - List file:line references for each violation
   - Suggest fixes where applicable

## Output Format

```
Lint Summary
============
gofmt:        PASS
golangci-lint: FAIL (3 issues)
tidb-custom:  PASS

Issues:
  pkg/executor/join.go:142: [golangci-lint] unused variable `ctx`
  pkg/planner/core/rule.go:87: [golangci-lint] error return value not checked
  util/chunk/chunk.go:310: [golangci-lint] S1039: unnecessary use of fmt.Sprintf
```

## Notes

- Requires `golangci-lint` to be installed; install via `make tools` or `go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest`
- The full `./...` lint may be slow on large diffs; prefer targeted package-level linting for PR reviews
- Lint failures should block merge but warnings are informational only

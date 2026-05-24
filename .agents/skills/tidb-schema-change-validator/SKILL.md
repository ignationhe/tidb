# tidb-schema-change-validator

Validates DDL schema change operations in TiDB pull requests to ensure correctness, backward compatibility, and adherence to TiDB's schema change best practices.

## Trigger

This skill is triggered when:
- A PR modifies files under `ddl/`, `meta/`, `infoschema/`, or `table/`
- A PR adds or modifies SQL test files containing DDL statements
- A PR description mentions schema changes, DDL, or table alterations

## What This Skill Does

1. **Detects schema change files** — Identifies modified Go files and SQL test files related to DDL operations
2. **Validates DDL compatibility** — Checks that schema changes are backward compatible with existing data
3. **Checks state machine transitions** — Validates that DDL job state transitions follow TiDB's online schema change protocol
4. **Verifies test coverage** — Ensures new DDL operations have corresponding integration tests
5. **Reviews error handling** — Confirms proper error handling for DDL rollback scenarios

## Inputs

- `pr_number` (required): The pull request number to validate
- `repo` (optional): Repository name, defaults to `pingcap/tidb`
- `strict_mode` (optional): Enable stricter compatibility checks, defaults to `false`

## Outputs

The skill produces a validation report containing:
- List of modified DDL-related files
- Compatibility warnings or errors
- Missing test coverage indicators
- State machine transition analysis
- Recommendations for improvement

## Validation Rules

### Critical (must fix)
- New DDL operations must implement `OnExceedMemQuota` if they hold locks
- Schema version bumps must be accompanied by version compatibility checks
- `SchemaDiff` structs must handle all new DDL types

### Warnings (should fix)
- New DDL job types should have cancellation support
- DDL operations affecting large tables should include progress reporting
- New column types should be tested with `SHOW CREATE TABLE` round-trip

### Informational
- Suggest adding DDL reorg worker tests for data-rewriting operations
- Remind to update `ddl/ddl_api.go` documentation for new public APIs

## Example Usage

```yaml
- uses: tidb-schema-change-validator
  with:
    pr_number: ${{ github.event.pull_request.number }}
    strict_mode: true
```

## Notes

- This skill integrates with TiDB's online DDL framework based on F1's schema change algorithm
- Validation respects TiDB's multi-state schema change protocol (none → delete-only → write-only → public)
- For questions about DDL internals, refer to `ddl/README.md` in the main repository

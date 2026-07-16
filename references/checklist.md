# Human-and-Agent CLI Checklist

Use this checklist during design and review. The rationale and normative rules
live in [CLI Design for Humans and Agents](cli-guidelines.md).

## Command shape and discovery

- [ ] Running the tool with no arguments gives concise help when input is required.
- [ ] Every command and subcommand supports `-h` and `--help`.
- [ ] Help shows purpose, required inputs, defaults, output modes, and examples.
- [ ] The most common human example appears before exhaustive options.
- [ ] At least one automation example uses the explicit structured mode.
- [ ] Related resources and operations use a consistent `noun verb` hierarchy.
- [ ] Verbs mean the same thing across resource types.
- [ ] There is no catch-all subcommand or arbitrary abbreviation.
- [ ] Invalid commands suggest likely corrections without silently executing them.
- [ ] Help links to version-matched or web documentation and a support path.

## Inputs and composition

- [ ] Positional arguments are unambiguous; otherwise descriptive flags are used.
- [ ] Every short flag has a long form.
- [ ] Standard flag names retain conventional meanings.
- [ ] `-o`/`--output` selects a destination; `--json`/`--format` selects a format.
- [ ] Flags are order-independent where the parser permits it.
- [ ] File input or output supports `-` for stdin/stdout where meaningful.
- [ ] Stdin consumption is explicit and documented.
- [ ] Required piped input fails quickly with help when stdin is a TTY.
- [ ] Bulk work accepts multiple arguments, selectors, manifests, or line input.
- [ ] Line input documents blank lines, duplicates, escaping, and invalid records.
- [ ] Secrets are accepted through safe files, stdin, sockets, or secret stores.
- [ ] Secrets are not required in flags or ordinary environment variables.

## Human output

- [ ] The default is concise human-readable text unless the tool transforms data.
- [ ] A successful mutation says what changed and shows the important new state.
- [ ] Long work responds quickly and shows TTY-aware progress.
- [ ] Default output excludes developer-only details and log-level prefixes.
- [ ] Color and symbols add meaning rather than decoration.
- [ ] Output remains understandable without color.
- [ ] Color honors non-TTY streams, `NO_COLOR`, `TERM=dumb`, and `--no-color`.
- [ ] Animation and paging are disabled when the relevant stream is not a TTY.
- [ ] `--quiet` reduces or suppresses output; it does not change the projection.

## Plain and structured output

- [ ] `--plain` exists when human formatting breaks line-oriented composition.
- [ ] Plain output has one stable undecorated record per line.
- [ ] A field selector exists when callers commonly need one bare identifier.
- [ ] Structured commands support explicit `--json`.
- [ ] JSON stdout contains valid JSON and nothing else.
- [ ] Diagnostics and progress go to stderr or are suppressed in JSON mode.
- [ ] Field names and types are stable and consistent.
- [ ] Sizes and durations use documented numeric base units.
- [ ] Timestamps have a documented format and timezone.
- [ ] Null, missing, empty, and zero values have distinct documented meanings.
- [ ] Long-lived structured contracts include a schema version.
- [ ] Streams use JSONL with exactly one complete object per line.
- [ ] JSONL records have a discriminator and a terminal record where practical.
- [ ] TTY detection never silently switches text output to JSON.

## Errors and exit status

- [ ] Expected errors say what failed and identify the failing input or resource.
- [ ] Errors say whether retrying is sensible.
- [ ] Errors suggest a concrete recovery action when one is known.
- [ ] Expected errors do not show raw tracebacks by default.
- [ ] Structured errors have stable symbolic identifiers.
- [ ] Structured errors echo relevant inputs using consistent field names.
- [ ] Success, including an idempotent no-op, returns `0`.
- [ ] Failure returns nonzero even when a structured error is emitted.
- [ ] Important exit statuses are documented and avoid shell-reserved values.
- [ ] Batch commands define atomic, best-effort, or transactional behavior.
- [ ] Partial failure identifies every failed resource and has defined exit behavior.

## Interactivity and safety

- [ ] Prompts occur only when stdin is a TTY.
- [ ] Every prompted value can also be supplied explicitly.
- [ ] `--no-input` disables all prompts and fails actionably if data is missing.
- [ ] Non-interactive invocation never hangs waiting for confirmation.
- [ ] Secret prompts disable terminal echo.
- [ ] `--no-input` does not imply consent.
- [ ] Moderate-risk actions support `--yes` or `--force`.
- [ ] Severe actions require exact confirmation such as `--confirm IDENTITY`.
- [ ] Hidden or cascading deletion is included in risk assessment.
- [ ] Complex or risky mutation supports `-n`/`--dry-run`.
- [ ] Dry-run uses the same discovery and policy logic as execution.
- [ ] Dry-run reports stable resource identities and structured planned changes.

## Idempotence and robustness

- [ ] Repeating a successful mutation converges without duplicates.
- [ ] Non-idempotent operations return a detectable conflict with existing identity.
- [ ] User input is validated before mutation where possible.
- [ ] External I/O has reasonable and configurable timeouts.
- [ ] Parallel output is serialized and never corrupts records.
- [ ] Transient failure can be retried without manual cleanup.
- [ ] Interruption leaves valid or recoverable state.
- [ ] Ctrl-C is acknowledged immediately and cleanup is bounded.
- [ ] Concurrent invocation and locking behavior are defined.
- [ ] A status or inspection command exposes non-obvious current state.

## Configuration, security, and privacy

- [ ] Configuration scope determines whether it uses flags, environment, or files.
- [ ] Precedence is documented: flags, environment, project, user, system.
- [ ] User configuration follows XDG conventions where appropriate.
- [ ] External or system configuration is not modified without consent.
- [ ] Standard environment variables such as `NO_COLOR` and `TMPDIR` are honored.
- [ ] `.env` is not used instead of typed project configuration.
- [ ] Environment variables do not contain secrets.
- [ ] The tool performs no hidden network activity.
- [ ] Telemetry is disclosed, consented to, and preferably opt-in.
- [ ] Installation is reversible and uninstall instructions are easy to find.

## Compatibility and testing

- [ ] Public subcommands, long flags, exit statuses, and schemas are inventoried.
- [ ] Machine contracts state whether unknown fields may be ignored.
- [ ] Changes are additive where possible.
- [ ] Breaking changes have warnings, replacements, and a deprecation period.
- [ ] Human presentation is not advertised as stable unless intentionally supported.
- [ ] Black-box tests cover no-argument help and recursive command discovery.
- [ ] Tests cover TTY and non-TTY presentation.
- [ ] Tests prove JSON stdout remains clean while diagnostics use stderr.
- [ ] Every JSONL line is parsed independently in tests.
- [ ] Plain output has compatibility fixtures.
- [ ] Tests cover every documented exit status and error identifier.
- [ ] Tests cover non-interactive prompt suppression and confirmation flags.
- [ ] Tests compare dry-run decisions with execution planning.
- [ ] Tests cover retry, idempotent no-op, conflict, and partial batch failure.
- [ ] Tests cover timeout, interruption, and concurrent invocation behavior.

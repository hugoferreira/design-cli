# CLI Design for Humans and Agents

## Status and intent

This document defines conventions for command-line interfaces used directly by
people, shell scripts, CI systems, and autonomous agents. It combines the
human-first principles of the [Command Line Interface Guidelines][clig] with
the stronger machine contracts required by agentic automation.

The goal is one coherent CLI, not separate human and agent products.

The key words **MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**, and **MAY**
describe requirement strength. Exceptions are allowed when the product has a
documented reason, but safety and contract requirements should not be weakened
for convenience.

[clig]: https://clig.dev/

## Core principles

1. **Humans come first by default.** Default output should be concise,
   readable, and responsive.
2. **Automation is a first-class mode.** Every operation that a human can
   perform interactively should have a complete non-interactive path.
3. **Semantics do not depend on presentation.** Text, plain, JSON, and JSONL
   modes may present results differently, but must represent the same command
   and resource decisions.
4. **TTY detection changes presentation and interaction, not intent.** It may
   control color, animation, paging, and prompting. It must not silently select
   different resources or mutate different state.
5. **Safe repetition is normal.** Agents retry and humans rerun commands.
   Mutations should be idempotent, convergent, or return a detectable conflict.
6. **Discovery is part of the interface.** A user or agent should be able to
   learn the command tree, required inputs, output modes, and recovery steps
   without searching source code.
7. **Machine formats are APIs.** Structured fields, plain records, exit codes,
   flags, and subcommands require explicit compatibility policies.
8. **Conventions beat novelty.** Standard names and familiar Unix behavior
   reduce learning cost for both people and agents.

## Interaction modes

A CLI should account for three modes without turning them into three different
products.

| Concern | Interactive TTY | Non-interactive text | Explicit structured mode |
|---|---|---|---|
| Primary output | Concise human text | Stable undecorated text | JSON or JSONL |
| Color and symbols | Allowed with intent | Disabled | Disabled |
| Progress | TTY-aware on stderr | No animation | Structured events or none |
| Paging | Allowed for long text | Disabled | Disabled |
| Prompts | Allowed when useful | Forbidden | Forbidden |
| Confirmation | Prompt according to risk | Require explicit flag | Require explicit field or flag |
| Errors | Actionable prose on stderr | Actionable prose on stderr | Stable error object plus nonzero exit |

The CLI MUST NOT automatically switch from text to JSON merely because stdout
is redirected. Agents can and should request JSON explicitly. Redirection may
disable decoration, but should not change the data model.

## Command design and discovery

### Command grammar

- A small tool MAY use a single command.
- A larger tool SHOULD use subcommands to group related behavior.
- Resource-oriented tools SHOULD use a consistent `noun verb` hierarchy, such
  as `service deploy`, `service inspect`, and `service delete`.
- The same operation SHOULD use the same verb across nouns.
- Similar or ambiguous verbs such as `update` and `upgrade` SHOULD NOT coexist
  without a clear distinction.
- Catch-all subcommands and implicit verbs SHOULD NOT be used. They make future
  expansion unsafe because a previously positional token can become a command.
- Arbitrary subcommand abbreviations MUST NOT be accepted. Explicit, permanent
  aliases are acceptable.

### Help and learning

Running the tool with no arguments SHOULD display concise help when arguments
are required. It should include:

- A one-sentence purpose.
- One or two common examples.
- The most important commands or flags.
- A pointer to `--help` for complete details.

Every command and subcommand MUST support `-h` and `--help`. A complex tool
SHOULD also support `help <command>`. Full help should state:

- Required and optional inputs.
- Defaults and valid values.
- Whether the command reads or changes state.
- Human, plain, and structured output options.
- Prompt and confirmation behavior.
- Relevant exit statuses.
- Realistic examples, including dry-run and automation.
- A documentation or support location.

When invalid input resembles a valid command or flag, the CLI SHOULD suggest a
correction but MUST NOT silently execute a state-changing interpretation.

## Inputs, arguments, and composition

### Arguments and flags

- Prefer flags when positional inputs would be ambiguous or likely to expand.
- Multiple positional paths are appropriate for simple file-oriented actions.
- Every short flag MUST have a descriptive long form.
- Reserve short flags for frequent, conventional behavior.
- Use established names where possible: `--json`, `--plain`, `--quiet`,
  `--dry-run`, `--force`, `--no-input`, `--version`, and `--help`.
- `-o` and `--output` SHOULD identify an output destination. Use `--json` or
  `--format` for format selection.
- Arguments and flags SHOULD be order-independent where parser capabilities
  allow it.

### Standard input and bulk work

- Commands dealing with file input or output SHOULD support `-` for stdin or
  stdout where meaningful.
- Stdin use MUST be explicit. A command must not unexpectedly consume a
  terminal or pipeline.
- If piped input is required but stdin is a TTY, the command SHOULD fail quickly
  with help instead of hanging.
- Bulk operations SHOULD be available through multiple arguments, selectors,
  manifests, or newline-delimited input. Automation should not require one
  process invocation per resource.
- Line-oriented input MUST document escaping, blank-line, duplicate, and error
  behavior.

### Secrets

Secrets MUST NOT be required in command arguments or ordinary environment
variables. They can leak through shell history, process listings, logs, and
diagnostics. Prefer credential files, stdin, local sockets, operating-system
keychains, or secret-management services.

## Output contracts

### Human-readable text

Human-readable text is the default unless the tool is inherently a data
transformer. Successful commands SHOULD print a brief result, especially after
changing state. They SHOULD avoid internal implementation details and log-level
prefixes.

Human output may use layout, grouping, color, and symbols when attached to a
TTY. It SHOULD:

- State what happened.
- Show important counts, identities, or new state.
- Highlight warnings without overwhelming the result.
- Suggest a sensible next command when part of a workflow.
- Remain useful when color is disabled.

Human output is allowed to evolve, unless the tool explicitly promises it as a
stable interface.

### Plain text

Tools that emit records SHOULD provide `--plain` when human formatting would
break line-oriented composition. Plain output MUST be stable and undecorated:

- One record per line.
- No headers unless explicitly requested.
- No color, animation, wrapping, or pager.
- Documented field order and delimiter.

If callers commonly need one identifier, provide a field selector such as
`--field name` rather than overloading `--quiet`.

### Quiet mode

`-q` and `--quiet` mean less output, not a different data projection. Quiet
mode SHOULD suppress non-essential success and progress messages. It SHOULD
NOT be used as an undocumented alias for “print names only.”

### JSON

Commands with structured results SHOULD support `--json`. In JSON mode:

- Stdout MUST contain valid JSON and nothing else.
- Diagnostics and progress MUST go to stderr or be suppressed.
- Field names and types MUST be consistent across invocations.
- Sizes and durations SHOULD be numeric base units, not formatted strings.
- Timestamps SHOULD use ISO 8601 with an explicit timezone, or documented Unix
  seconds where arithmetic is primary.
- Missing, null, empty, and zero values MUST have documented meanings.
- The response SHOULD include a schema or contract version when long-lived
  automation is expected.
- Nested structures are appropriate for real hierarchy, but accidental wrapper
  depth should be avoided.

### JSON Lines

Long-running and streaming commands SHOULD support JSONL when incremental
consumption is useful. JSONL output MUST contain exactly one complete JSON
object per line. Records SHOULD have a discriminator such as `record_type` and
the stream SHOULD end with a summary or terminal record when practical.

### Stdout and stderr

- Primary results belong on stdout.
- Human diagnostics, warnings, progress, and errors belong on stderr.
- In explicit structured mode, a machine-readable error object MAY be written
  to stdout so callers can parse it, but the command MUST also return nonzero.
- A structured error convention must be consistent across every subcommand.
- No animation should be emitted when its stream is not a TTY.
- Color should be disabled when the relevant stream is not a TTY, when
  `NO_COLOR` is set, when `TERM=dumb`, or when `--no-color` is passed.

## Errors and exit status

Every expected error should answer four questions:

1. What failed?
2. Which input or resource was involved?
3. Is retrying sensible?
4. What can the caller do next?

Human errors SHOULD be concise, specific, and actionable. Expected errors MUST
NOT expose raw tracebacks by default. Unexpected errors SHOULD provide a way to
obtain debug details and report a bug.

Structured errors SHOULD include:

```json
{
  "ok": false,
  "error": "resource_not_found",
  "message": "service 'web' does not exist",
  "resource_name": "web",
  "retryable": false,
  "suggestion": "run 'myctl service list --json'"
}
```

Exit status is control flow:

- `0` means the requested operation succeeded, including an idempotent no-op.
- Nonzero means the requested operation did not fully succeed.
- Usage errors SHOULD use `2` where that convention fits the platform.
- Important failure classes MAY receive distinct documented statuses.
- Avoid statuses reserved by the shell, including `126`, `127`, and values
  representing signals.
- Symbolic structured error identifiers are more portable than assuming every
  CLI shares one universal numeric taxonomy.

Batch commands MUST define whether they are atomic, best-effort, or
transactional. Partial success must return nonzero unless the command explicitly
defines partial completion as success, and must identify each failed resource.

## Interactivity and safety

### Prompts

- Prompts MAY be used only when stdin is a TTY.
- A prompt MUST never be the only way to provide required input.
- `--no-input` MUST disable every prompt. Missing data should then produce an
  actionable error.
- Non-interactive invocation MUST fail rather than hang waiting for input.
- Secret prompts must disable terminal echo.

`--no-input` and confirmation flags serve different purposes:

- `--no-input` says “do not interact.”
- `--yes`, `--force`, or `--confirm VALUE` says “the caller authorizes this
  risk.”

One must not imply the other.

### Risk-based confirmation

Use the operation’s real blast radius rather than a universal confirmation
rule.

| Risk | Examples | Interactive behavior | Automation behavior |
|---|---|---|---|
| Mild | Explicit local edit, reversible update | Usually no prompt | Execute directly |
| Moderate | Bulk local mutation, remote deletion | Show scope and confirm | Require `--yes` or `--force` |
| Severe | Irreversible production deletion | Require exact typed identity | Require `--confirm IDENTITY` |

Hidden or indirect deletion counts toward the risk level. An explicit command
named `delete` does not make an unexpectedly large cascade safe.

### Dry-run

Commands that perform complex or risky changes SHOULD support `-n` and
`--dry-run`. Dry-run must execute the same discovery and policy logic as the
real command while suppressing mutation. It should report stable resource
identities and structured planned changes, not merely a prose promise.

## Idempotence, robustness, and recovery

- Prefer declarative `apply`, `ensure`, and `sync` semantics when they match the
  domain.
- Repeating a successful command SHOULD converge without introducing duplicate
  state.
- If idempotence is impossible, return a distinct conflict that exposes the
  existing resource identity.
- Validate all user input before mutation where possible.
- Long operations SHOULD provide responsive progress on a TTY and configurable
  timeouts for external I/O.
- Parallel output MUST be serialized so records and messages do not interleave.
- Transient failures SHOULD be retryable without manual cleanup.
- Crash-only design is preferred: interruption should leave either valid state
  or enough durable information for the next run to recover.
- Ctrl-C should be acknowledged immediately and terminate promptly. Cleanup
  must be bounded and a second interrupt should be able to skip it.
- State-changing tools SHOULD provide an inspection or status command.
- Concurrent invocation behavior and locking MUST be defined.

## Configuration and environment

Use configuration according to its scope:

1. Per-invocation values: flags.
2. Session or machine context: environment variables and user configuration.
3. Project-wide stable values: version-controlled project configuration.
4. System policy: system configuration.

Precedence should be explicit and normally follow:

1. Flags.
2. Environment variables.
3. Project configuration.
4. User configuration.
5. System configuration.

User configuration SHOULD follow the XDG Base Directory Specification where
the platform supports it. A CLI MUST obtain consent before modifying another
program’s or the system’s configuration.

Environment variables should be uppercase, portable, and single-line. Honor
standard variables such as `NO_COLOR`, `TERM`, `PAGER`, and `TMPDIR` where
relevant. Do not use `.env` as a substitute for typed, versioned project
configuration, and do not place secrets in environment variables.

## Evolution and compatibility

Treat these as public interfaces:

- Command and subcommand names.
- Long flags and argument meaning.
- Exit statuses promised by documentation.
- Plain output field order.
- JSON and JSONL schemas.
- Configuration files and environment variables.

Changes SHOULD be additive. Breaking changes require a deprecation period,
runtime warning, documented replacement, and a way for callers to migrate
before removal.

Human presentation may evolve to improve usability. Documentation should tell
scripts and agents to select `--plain`, `--json`, or JSONL rather than parsing
decorated text.

Schemas should be versioned deliberately. Adding optional fields is generally
safe; renaming fields, changing types, or changing nullability is not. Consumers
should be told whether unknown fields may be ignored.

## Naming, distribution, and telemetry

- Command names should be short, memorable, lowercase, and easy to type.
- Prefer a single binary or the platform’s normal package manager.
- Installation should be reversible and uninstall instructions should be easy
  to find.
- The tool must not perform hidden network activity.
- Usage or crash telemetry must not be sent without clear disclosure and
  consent. Opt-in collection is preferred.
- Before collecting telemetry, consider documentation analytics, download
  counts, issue reports, and direct user research.

## Conformance expectations

A production CLI should test its interface from the outside, not only its
internal functions. At minimum, automated tests should cover:

- No-argument and recursive help discovery.
- TTY and non-TTY presentation behavior.
- Clean JSON stdout with diagnostics isolated to stderr.
- Every JSONL line parsing independently.
- Plain output record stability.
- Documented exit statuses and structured error identifiers.
- Prompt suppression with `--no-input` and non-TTY stdin.
- Confirmation requirements at each risk level.
- Dry-run equivalence to execution planning.
- Idempotent retry and conflict behavior.
- Partial batch failures.
- Ctrl-C and timeout behavior.
- Compatibility fixtures for structured schemas.

## Decision checklist

Before adding a command, answer:

1. What is the human default?
2. What exact flag selects the stable machine contract?
3. What changes between TTY and non-TTY execution?
4. Can every prompt be replaced by explicit input?
5. What is the operation’s risk level and confirmation mechanism?
6. What happens when the command is retried or interrupted?
7. How are partial failures represented?
8. Which outputs and statuses are compatibility promises?
9. How will a new user and an agent discover the command?
10. What black-box tests prove the contract?

## Sources and scope

This document adapts the complete [Command Line Interface Guidelines][clig],
including its guidance on human-first output, help, errors, arguments,
interactivity, subcommands, robustness, compatibility, signals, configuration,
environment variables, naming, distribution, and analytics. It extends that
foundation with explicit schemas, JSONL streaming, retry semantics, structured
errors, batch behavior, and conformance requirements for autonomous agents.

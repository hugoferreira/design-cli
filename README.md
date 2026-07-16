# Design CLIs

`design-clis` is a Codex skill for designing, reviewing, and refactoring
command-line interfaces that work well for both people and automation.

It treats human-readable output as the default experience while making
machine-facing behavior explicit and dependable: structured output, stream
separation, exit statuses, non-interactive operation, retries, and
compatibility are all part of the public interface.

## What it covers

- Command trees, arguments, flags, and help text
- Human, plain, quiet, JSON, and JSONL output modes
- `stdout`, `stderr`, and exit-status contracts
- Errors, prompts, confirmations, and dry runs
- Batch operations, partial failures, retries, and idempotence
- Configuration precedence and compatibility policy
- Black-box tests at the process boundary

## Use the skill

Ask Codex to use `$design-clis` when creating a CLI contract, auditing an
existing tool, planning a refactor, or implementing and testing an interface.

Example prompts:

```text
Use $design-clis to design the command tree and output contract for this tool.

Use $design-clis to audit this CLI for scripting and autonomous-agent use.

Use $design-clis to add a stable JSON mode and black-box contract tests.
```

When reviewing an existing CLI, give Codex access to the executable or source
tree so it can inspect help and exercise success, empty, warning, and failure
paths.

## Design approach

The skill follows a five-part workflow:

1. Inspect the existing interface from the user's point of view.
2. Define separate contracts for interactive use, shell composition,
   autonomous invocation, and streaming work.
3. Reconcile those contracts around a humane default and explicit machine
   modes.
4. Produce an actionable design, review, or cohesive implementation.
5. Verify exact output streams and exit statuses from the process boundary.

Its central principle is simple: keep the default concise and readable for
people, while treating machine formats and control-flow behavior as stable
APIs. TTY detection may alter presentation and prompt eligibility, but never
the meaning of a command or its schema.

## Repository contents

| Path | Purpose |
| --- | --- |
| [`SKILL.md`](SKILL.md) | Skill trigger, workflow, and operating instructions |
| [`references/guidelines.md`](references/guidelines.md) | Normative human-and-agent CLI design guidance |
| [`references/checklist.md`](references/checklist.md) | Audit, planning, and release-readiness checklist |
| [`references/contract-example.md`](references/contract-example.md) | Worked CLI contract with output, error, safety, and test examples |
| [`agents/openai.yaml`](agents/openai.yaml) | Display metadata and default invocation prompt |

Start with [`SKILL.md`](SKILL.md). Before making design decisions, the skill
loads the normative guidelines and then uses the checklist or worked example
when the task calls for them.

## Expected deliverables

Depending on the request, the skill can produce:

- A consistent noun-verb command tree
- Representative help and output examples
- Structured field and type contracts
- Exit-status and symbolic error taxonomies
- Prompt, consent, dry-run, and non-interactive rules
- Compatibility and migration guidance
- Process-level acceptance tests

The skill favors compatible, incremental improvements for established CLIs
and clearly distinguishes observed behavior from proposed behavior.

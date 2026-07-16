---
name: design-clis
description: Design, review, or refactor command-line interfaces for both interactive people and autonomous agents. Use for CLI command trees, arguments and flags, help text, human output, JSON or JSONL contracts, plain and quiet modes, stdout and stderr behavior, exit codes, errors, prompts, confirmations, dry runs, idempotence, batch operations, configuration, compatibility, or black-box CLI tests.
---

# Design CLIs

Create command-line interfaces that remain pleasant for people while exposing explicit, stable contracts for automation. Treat human-readable output as the default and make machine-readable modes discoverable and dependable.

## Load the right references

Read `references/guidelines.md` before making design decisions or reviewing an existing CLI. It is the normative guidance and reconciles clig.dev's human-first principles with agent-era requirements.

Also read the following when relevant:

- `references/checklist.md` for audits, implementation plans, acceptance criteria, or release readiness.
- `references/contract-example.md` when defining a concrete command contract, implementing output modes, or writing black-box tests.

Do not rely on excerpts in this file in place of the normative guidance.

## Follow the workflow

### 1. Inspect the existing interface

Use the CLI itself when it exists. Run the top-level and relevant subcommand help, then exercise representative success, empty, warning, and failure cases. Inspect behavior with a TTY and without one when interactivity or presentation can differ.

Record:

- command hierarchy and naming grammar;
- input channels, batch capabilities, and precedence;
- human, plain, quiet, JSON, and streaming output modes;
- stdout, stderr, and exit-status behavior;
- prompts, confirmations, dry runs, and non-interactive behavior;
- retry, idempotence, partial-failure, and compatibility behavior.

Distinguish observed behavior from proposed behavior.

### 2. Define audiences and contracts

Identify interactive human use, shell composition, autonomous invocation, and long-running or streaming use. Do not force one presentation to serve every audience.

Define these contracts explicitly:

- command and argument grammar;
- input and configuration precedence;
- default human presentation;
- opt-in structured and plain projections;
- stdout and stderr separation;
- stable field names and types;
- error identifiers and exit statuses;
- prompt and consent rules;
- idempotence, retries, batch outcomes, and partial failure;
- compatibility and deprecation policy.

### 3. Reconcile human and agent needs

Apply these invariants while using the full guidance for details:

- Keep concise, readable text as the default output.
- Provide explicit `--json` or equivalent structured output; use JSONL for incremental records.
- Keep machine-readable stdout clean. Send diagnostics and progress to stderr.
- Let TTY detection change presentation and prompt eligibility, never result meaning or schema.
- Separate reduced output (`--quiet`) from stable bare-value projection (`--plain` or `--field`).
- Separate output destination (`--output`) from representation (`--format`) when both exist.
- Disable prompts explicitly with `--no-input`; require a separate consent flag such as `--yes` when risk needs authorization.
- Prefer declarative, idempotent operations. Make unavoidable conflicts and retryable failures programmatically distinguishable.
- Make help complete enough for deterministic discovery, including defaults, required inputs, output modes, exit statuses, and realistic examples.

Do not make JSON the default merely because an agent may invoke the command. Agents can select a documented machine mode; people should receive a humane default.

### 4. Produce an actionable design

For a new CLI, provide:

1. a noun-verb command tree;
2. representative help text;
3. success, empty, warning, and error examples for each relevant output mode;
4. a field and type contract for structured output;
5. an exit-status and error taxonomy;
6. prompt, dry-run, consent, and non-interactive rules;
7. compatibility and testing requirements.

For an existing CLI review, rank findings by automation breakage, safety risk, human usability, and migration cost. Preserve compatible behavior where possible and propose staged migrations for breaking changes.

For implementation work, make small cohesive changes. Update help, behavior, tests, and documentation together rather than leaving the contract implicit.

### 5. Verify from the process boundary

Test the compiled or installed CLI as a user or agent would invoke it. Assert exact stdout, stderr, and exit status separately. Cover the matrix in `references/checklist.md`, including:

- TTY and non-TTY execution;
- text, plain, quiet, JSON, and JSONL modes as supported;
- success, empty results, warnings, usage errors, domain errors, and partial failures;
- interrupted and repeated operations;
- prompts with and without explicit consent;
- schema and help compatibility.

Prefer black-box tests for public contracts. Supplement them with unit tests, but do not substitute internal assertions for process-boundary verification.

## Report decisions clearly

Lead with the proposed or implemented contract. State tradeoffs, compatibility implications, and unresolved risks. When guidance conflicts with established project conventions, explain the conflict and choose deliberately rather than silently mixing semantics.

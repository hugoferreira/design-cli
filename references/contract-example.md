# Reference CLI Contract: `myctl`

This worked example applies [CLI Design for Humans and Agents](cli-guidelines.md)
to a fictional deployment tool. It is a contract example, not a prescription
for every product domain.

## Command tree

```text
myctl
├── service
│   ├── list
│   ├── inspect
│   ├── deploy
│   ├── delete
│   └── events
├── environment
│   ├── list
│   └── inspect
└── auth
    ├── login
    └── status
```

The grammar is consistently `noun verb`. There is no implicit deployment
command and no arbitrary abbreviation of verbs.

## Top-level discovery

Running `myctl` without arguments prints concise help:

```text
myctl manages services in deployment environments.

Examples:
  myctl service list --environment staging
  myctl service deploy web --environment production

Commands:
  service       List, inspect, deploy, and delete services
  environment   List and inspect environments
  auth          Manage authentication

Run 'myctl <command> --help' for complete help.
```

`myctl service deploy --help` includes required inputs, defaults, human and
structured output options, dry-run behavior, confirmation rules, exit statuses,
and examples:

```text
Deploy a service to an environment. Repeating the same deployment is a no-op.

Usage:
  myctl service deploy SERVICE --environment ENVIRONMENT [flags]

Arguments:
  SERVICE                    Service name (required)

Required flags:
  --environment ENVIRONMENT Target environment

Common flags:
  --image IMAGE              Image override (default: service configuration)
  -n, --dry-run              Report planned changes without deployment
  --no-input                 Disable all prompts
  --yes                      Confirm a production deployment non-interactively
  --timeout DURATION         Maximum wait (default: 5m)
  --json                     Emit one JSON result
  --format jsonl             Stream JSON Lines events
  --plain                    Emit stable line-oriented text
  -q, --quiet                Suppress non-essential output
  -o, --output FILE          Write the primary result to FILE; '-' means stdout

Examples:
  myctl service deploy web --environment staging
  myctl service deploy web --environment production --dry-run
  myctl service deploy web --environment production --yes --json
```

## Human default

Human-readable text is the default:

```console
$ myctl service deploy web --environment staging
Deploying web to staging…
  Image:      registry.example/web:2.1.0
  Instances:  3
  Revision:   42 → 43

Deployed web revision 43 in 18.4s.
Run 'myctl service inspect web --environment staging' to view its status.
```

Progress is written to stderr and is animated only when stderr is a TTY. The
final result is written to stdout. `--quiet` suppresses progress and the final
success explanation; it does not print a different field.

## Plain output

`service list` offers stable line output for shell composition:

```console
$ myctl service list --environment staging --plain
api	Running	42
web	Running	43
worker	Failed	18
```

The contract documents the fields as `name`, `status`, and `revision`, separated
by tabs, one service per line. Callers needing one field use `--field`:

```console
$ myctl service list --environment staging --field name
api
web
worker
```

This avoids redefining `--quiet` as “names only.”

## JSON result

An agent requests JSON explicitly:

```console
$ myctl service deploy web --environment staging --json
{
  "schema_version": 1,
  "ok": true,
  "command": "service deploy",
  "changed": true,
  "service_name": "web",
  "environment_name": "staging",
  "image": "registry.example/web:2.1.0",
  "previous_revision": 42,
  "revision": 43,
  "instance_count": 3,
  "elapsed_seconds": 18.4
}
```

Rules demonstrated here:

- Stdout contains one JSON document and nothing else.
- Durations and counts are numeric.
- Fields are relatively flat because the result has no meaningful deep
  hierarchy.
- `changed` distinguishes an applied deployment from an idempotent no-op.
- Progress and diagnostics do not contaminate stdout.

Repeating the same deployment succeeds without mutation:

```json
{
  "schema_version": 1,
  "ok": true,
  "command": "service deploy",
  "changed": false,
  "service_name": "web",
  "environment_name": "staging",
  "image": "registry.example/web:2.1.0",
  "previous_revision": 43,
  "revision": 43,
  "instance_count": 3,
  "elapsed_seconds": 0.3
}
```

Both invocations return exit status `0`.

## JSONL stream

Long deployments can stream events:

```console
$ myctl service deploy web --environment staging --format jsonl
{"schema_version":1,"record_type":"phase","phase":"resolve","status":"complete"}
{"schema_version":1,"record_type":"phase","phase":"rollout","status":"started"}
{"schema_version":1,"record_type":"instance","instance_name":"web-7f9c-1","status":"ready"}
{"schema_version":1,"record_type":"instance","instance_name":"web-7f9c-2","status":"ready"}
{"schema_version":1,"record_type":"instance","instance_name":"web-7f9c-3","status":"ready"}
{"schema_version":1,"record_type":"summary","ok":true,"changed":true,"service_name":"web","revision":43}
```

Each line is independently valid JSON. `record_type` identifies its schema and
the final summary terminates the stream.

## Structured errors

If an image is missing, human mode prints to stderr:

```text
Image 'registry.example/web:2.1.0' was not found.
Check the tag or run 'myctl service inspect web --environment staging'.
```

It produces no stdout and returns `3`, the command’s documented not-found
status.

With `--json`, stdout contains the structured error, stderr may contain the
brief human explanation, and the status remains nonzero:

```json
{
  "schema_version": 1,
  "ok": false,
  "error": "image_not_found",
  "message": "image 'registry.example/web:2.1.0' was not found",
  "image": "registry.example/web:2.1.0",
  "registry": "registry.example",
  "retryable": false,
  "suggestion": "check the image tag or inspect the service configuration"
}
```

The caller can use the exit status for broad control flow and `error` for the
specific recovery path.

## Exit statuses

`myctl` publishes this contract:

| Status | Meaning |
|---:|---|
| 0 | Success, including an idempotent no-op |
| 1 | General or partial operation failure |
| 2 | Invalid usage or missing non-interactive confirmation |
| 3 | Requested resource not found |
| 4 | Authentication or authorization failure |
| 5 | State conflict |
| 6 | Unsupported capability |
| 75 | Temporary remote failure; retry may succeed |
| 130 | Interrupted by Ctrl-C |

These values are part of `myctl`’s contract, not a claim that every CLI uses the
same numeric mapping. Structured error identifiers remain the more specific and
portable interface.

## Dry-run contract

Dry-run uses the same resolver, policy, authorization checks, and diff engine as
execution. It does not write state:

```console
$ myctl service deploy web --environment production --dry-run --json
{
  "schema_version": 1,
  "ok": true,
  "command": "service deploy",
  "dry_run": true,
  "changed": false,
  "would_change": true,
  "service_name": "web",
  "environment_name": "production",
  "current_revision": 42,
  "planned_revision": 43,
  "changes": [
    {
      "change_type": "replace",
      "field": "image",
      "before": "registry.example/web:2.0.0",
      "after": "registry.example/web:2.1.0"
    }
  ],
  "warnings": [
    "the rollout will restart 3 instances"
  ]
}
```

`changed` describes actual mutation and remains false. `would_change` describes
the plan. This avoids overloading one field with two meanings.

## Prompts and confirmation

Staging deployment is mild risk and does not prompt. Production deployment is
moderate risk.

When stdin is a TTY:

```console
$ myctl service deploy web --environment production
Deploy web revision 43 to production and restart 3 instances? [y/N]
```

When stdin is not a TTY, the same invocation fails immediately with status `2`:

```text
Production deployment requires confirmation in non-interactive mode.
Pass '--yes', or preview with '--dry-run'.
```

`--no-input` also disables the prompt and produces that error. It does not
authorize the deployment. `--yes` supplies authorization:

```bash
myctl service deploy web --environment production --no-input --yes --json
```

Severe deletion requires the exact environment and service identity:

```bash
myctl service delete web \
  --environment production \
  --confirm production/web \
  --json
```

## Batch input and partial failure

Batch deployment reads an explicit manifest, with `-` meaning stdin:

```bash
generate-deployments |
  myctl service deploy-batch --file - --environment staging --format jsonl
```

The command is best-effort rather than atomic. It emits one result per service
and a summary. If any service fails, the final status is `1` even when other
services succeeded:

```json
{"schema_version":1,"record_type":"service","service_name":"api","ok":true,"changed":false}
{"schema_version":1,"record_type":"service","service_name":"web","ok":true,"changed":true}
{"schema_version":1,"record_type":"service","service_name":"worker","ok":false,"error":"image_not_found"}
{"schema_version":1,"record_type":"summary","ok":false,"succeeded":2,"failed":1}
```

Retrying the same manifest leaves `api` and `web` unchanged and retries
`worker`.

## Secrets

Authentication accepts tokens from a credential file, stdin, the operating
system keychain, or an agent socket. It does not accept `--token VALUE` and does
not require a token environment variable:

```bash
myctl auth login --token-file ~/.config/myctl/token
secret-tool lookup service myctl | myctl auth login --token-file -
```

Interactive secret prompts disable echo. `--no-input` requires an explicit
credential source.

## Compatibility policy

`myctl` treats the following as stable interfaces:

- Command and long flag names.
- Exit statuses listed above.
- Plain output field order.
- JSON and JSONL field names, types, and record discriminators.
- Configuration keys and precedence.

Within schema version 1, consumers must ignore unknown fields. New optional
fields may be added. Existing fields cannot be renamed, removed, made required,
or assigned a different type without a new schema version and deprecation
period.

Human output is intentionally evolvable. Scripts are instructed to use
`--plain`, `--json`, or `--format jsonl`.

## Black-box acceptance tests

The release suite executes the installed binary and verifies:

1. `myctl`, every noun, and every verb expose useful help.
2. Human mode is readable and does not accidentally emit JSON.
3. JSON stdout parses as one document and contains no diagnostics.
4. Every JSONL line parses independently and the stream ends in a summary.
5. Plain output matches a versioned fixture.
6. Human errors use stderr; JSON errors preserve nonzero status.
7. Every documented status has a reproducible scenario.
8. Non-TTY commands never prompt.
9. `--no-input` refuses missing values and `--yes` supplies consent separately.
10. Dry-run and execution generate the same resource plan.
11. Repeating a deployment produces an idempotent no-op.
12. Batch partial failure identifies all resources and returns nonzero.
13. Ctrl-C exits promptly and a retry resumes safely.
14. Concurrent invocations either serialize safely or return a conflict.

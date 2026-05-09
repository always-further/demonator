# Hidden Steps

A hidden step runs a command silently — no prompt is shown, no text is typed,
and no keypress is required. The command executes immediately and its output can
be captured into a variable for use in later steps.

## Basic usage

```yaml
steps:
  - text: "nono audit list --recent 1 --json"
    hidden: true
    capture:
      name: session_id
      json_path: "[0].session_id"

  - text: "nono audit show {session_id}"
```

The first step runs invisibly and stores the session ID. The second step types
and executes normally, with `{session_id}` substituted in.

## When to use hidden steps

Use `hidden: true` when a command is infrastructure for the demo rather than
part of the story — for example, fetching an ID or token that a later step
needs, without cluttering the screen with the lookup command.

For commands that should run before the demo starts entirely, prefer
[setup commands](setup-teardown.md) instead. Hidden steps are useful when the
command must run at a specific point in the demo flow (e.g., after a previous
step has created a resource).

## Capturing JSON output

The `json_path` capture option is particularly useful with hidden steps:

```yaml
steps:
  - text: "myapp jobs create --json"
    hidden: true
    capture:
      name: job_id
      json_path: "id"

  - text: "myapp jobs wait {job_id}"
  - text: "myapp jobs logs {job_id}"
```

See [Configuration](configuration.md#capture-block) for the full `json_path`
syntax reference.

## How it works

When `hidden: true` is set:

- The prompt is not printed
- The command text is not typed
- No keypress or auto-advance wait occurs
- The command executes immediately
- Any `capture` block is still applied and the variable is stored
- The step is otherwise invisible to the audience

## Dry-run

Hidden steps are shown in `--dry-run` output with a `[hidden]` annotation so
you can verify they are in the right position:

```
~ $ nono audit list --recent 1 --json  [hidden, capture]
~ $ nono audit show {session_id}
```

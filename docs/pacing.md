# Pacing & Manual Control

By default, demonator types a command and then waits for you to press **Enter**
before executing it. Two per-step flags give you finer control over where those
manual pauses happen.

## `wait_before`

Show the prompt and pause for Enter *before* the command starts typing. Useful
when you want to speak to the audience before the next command appears.

```yaml
steps:
  - text: "cargo build --release"

  - text: "cargo test"
    wait_before: true   # pause here before typing starts
```

Flow:

```
~ $ cargo build --release
<output>
[you press Enter]           ← wait_before pause (cursor is at the prompt)
~ $ cargo test              ← typing begins
[you press Enter]           ← normal execution pause
<output>
```

## `wait_after`

Pause for Enter *after* the command finishes and its output is shown, before
the next prompt appears. Useful when you want time to explain the output before
moving on.

```yaml
steps:
  - text: "kubectl get pods"
    wait_after: true    # pause here after output is shown

  - text: "kubectl describe pod my-pod"
```

Flow:

```
~ $ kubectl get pods
<output>
[you press Enter]           ← wait_after pause
~ $ kubectl describe pod my-pod
[you press Enter]           ← normal execution pause
<output>
```

## Combining both

You can set both flags on the same step to bracket it with pauses:

```yaml
steps:
  - text: "terraform apply"
    wait_before: true   # let you explain before typing
    wait_after: true    # let you explain the output
```

## Both flags vs `pause` directive

The `pause` directive shows a bare prompt and waits for Enter without running
any command:

```yaml
steps:
  - text: "echo hello"
  - pause              # just waits — no command typed or run
  - text: "echo world"
```

Use `pause` when you need a mid-demo break that isn't tied to a specific
command. Use `wait_before` / `wait_after` when you want the pause anchored to a
particular step.

## Dry-run display

In `--dry-run` mode, steps with these flags show the appropriate annotation:

```
~ $ cargo build --release
~ $ cargo test  [wait-before]
~ $ kubectl get pods  [wait-after]
~ $ terraform apply  [wait-before, wait-after]
```

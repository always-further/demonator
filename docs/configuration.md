# Configuration Reference

Demonator is configured with a YAML file (default: `demo.yml`).

## Global options

| Option         | Type     | Default                              | Description                                      |
|----------------|----------|--------------------------------------|--------------------------------------------------|
| `prompt`       | string   | `{green}~{reset} {blue}${reset} `   | Prompt displayed before each command              |
| `clear`        | bool     | `false`                              | Clear terminal before starting                    |
| `speed`        | integer  | --                                   | Characters per second (overrides `delay`)         |
| `delay`        | integer  | `50`                                 | Milliseconds between characters                   |
| `jitter`       | integer  | `40`                                 | Random timing variation as a percentage           |
| `pause`        | integer  | `200`                                | Extra milliseconds after punctuation              |
| `highlight`    | bool     | `false`                              | Syntax-highlight commands as they are typed       |
| `auto_advance` | integer  | --                                   | Auto-advance delay in ms (skips Enter prompt)     |
| `setup`        | string[] | --                                   | Commands to run silently before the demo          |
| `teardown`     | string[] | --                                   | Commands to run silently after the demo           |
| `env`          | map      | `{}`                                 | Environment variables exported into every command |

## Steps

A demo is defined as a list of steps. Each step is one of:

### Command step

```yaml
- text: "echo hello"
  speed: 30           # override global speed
  delay: 40           # override global delay
  jitter: 20          # override global jitter
  pause: 100          # override global pause
```

If both `speed` and `delay` are set on the same step, `speed` wins.

### Directive

Simple string steps that control flow:

```yaml
- pause    # show prompt, wait for Enter, don't run anything
- clear    # clear the terminal screen
```

### Comment

Styled text that appears without a prompt:

```yaml
- comment: "Explanatory text here"
  style: dim           # optional: dim, bold, italic, or a color name
```

## Per-step command options

These fields are available on command steps (`text:` steps):

| Field          | Type     | Default | Description                                         |
|----------------|----------|---------|-----------------------------------------------------|
| `text`         | string   | *required* | The command to type and execute                  |
| `speed`        | integer  | --      | Characters per second for this step                 |
| `delay`        | integer  | --      | Milliseconds between characters for this step       |
| `jitter`       | integer  | --      | Timing variation percentage for this step           |
| `pause`        | integer  | --      | Extra punctuation pause for this step               |
| `capture`      | object   | --      | Capture output with regex (see below)               |
| `fake_output`  | string   | --      | Pre-defined output to show                          |
| `output_speed` | integer  | --      | Characters per second for typing fake output        |
| `execute`      | bool     | `true`  | Whether to actually run the command                 |
| `wait_for`     | string   | --      | Regex pattern to wait for in output                 |
| `timeout`      | integer  | `30`    | Seconds before wait_for gives up                    |
| `wait`         | integer  | --      | Per-step auto-advance override (ms)                 |
| `interact`     | object[] | --      | Expect-style interaction pairs                      |
| `if`           | string   | --      | Only run if this captured variable is set           |
| `unless`       | string   | --      | Only run if this captured variable is NOT set       |
| `wait_before`  | bool     | `false` | Pause for Enter before typing starts (see [Pacing](pacing.md)) |
| `wait_after`   | bool     | `false` | Pause for Enter after output is shown (see [Pacing](pacing.md)) |
| `env`          | map      | `{}`    | Per-step env vars (merged over global, step wins)   |

## Environment variables

Set env vars at the global level to apply to every command, every `setup`/
`teardown` hook, and every `wait_for`/`interact` invocation:

```yaml
env:
  AWS_PROFILE: demo
  LOG_LEVEL: debug

steps:
  - text: "aws s3 ls"          # sees AWS_PROFILE=demo
```

Override or extend for a single step with a per-step `env:`. Per-step keys
win over global keys; other global keys are inherited:

```yaml
env:
  LOG_LEVEL: debug

steps:
  - text: "./run --verbose"    # LOG_LEVEL=debug
  - text: "./run --quiet"
    env:
      LOG_LEVEL: error         # this step only
```

Values are passed verbatim to the spawned shell — they are not interpolated
by demonator. To pull a value from the host environment, reference it from
the command itself: `text: "echo $TOKEN"`.

## Capture block

```yaml
capture:
  name: my_var           # variable name for later substitution
  pattern: "regex (\\w+)" # regex with one capture group
```

The first capture group is extracted from combined stdout+stderr and stored.
Reference it in later steps with `{my_var}`.

## Interact block

```yaml
interact:
  - expect: "prompt text"    # string to match in stdout
    send: "response"         # text to send (newline appended automatically)
```

## Chapters

An alternative to flat `steps`. Cannot be used together with `steps`.

```yaml
chapters:
  - name: "Section Name"
    steps:
      - text: "command here"
```

## Prompt colors

Available color placeholders for the `prompt` string:

| Placeholder  | Effect    |
|-------------|-----------|
| `{black}`   | Black     |
| `{red}`     | Red       |
| `{green}`   | Green     |
| `{yellow}`  | Yellow    |
| `{blue}`    | Blue      |
| `{magenta}` | Magenta   |
| `{cyan}`    | Cyan      |
| `{white}`   | White     |
| `{bold}`    | Bold      |
| `{dim}`     | Dim       |
| `{reset}`   | Reset     |

## CLI flags

| Flag         | Description                                    |
|--------------|------------------------------------------------|
| `-c, --config` | Path to config file (default: `demo.yml`)    |
| `--dry-run`  | Preview demo flow without executing            |
| `--watch`    | Re-run demo when config file changes           |

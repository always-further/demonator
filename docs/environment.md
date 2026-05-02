# Environment Variables

Demonator can export environment variables into every shell it spawns —
commands, `setup`/`teardown` hooks, `wait_for` runs, and `interact` runs all
see them.

## Global env

Set `env` at the top level. Every step inherits these values:

```yaml
env:
  AWS_PROFILE: demo
  LOG_LEVEL: debug
  API_BASE_URL: http://localhost:8080

steps:
  - text: "aws s3 ls"          # AWS_PROFILE=demo
  - text: "curl $API_BASE_URL/health"
```

## Per-step env

Add `env:` to a single command step to override or extend the global map. The
merge is per-key: keys you define on the step win; keys you do not redefine
are still inherited from global:

```yaml
env:
  LOG_LEVEL: debug

steps:
  - text: "./run --verbose"          # LOG_LEVEL=debug
  - text: "./run --quiet"
    env:
      LOG_LEVEL: error               # this step only
  - text: "./run --verbose again"    # back to LOG_LEVEL=debug
```

## Values are not interpolated

Demonator passes env values to the spawned shell verbatim. Writing
`TOKEN: ${HOST_TOKEN}` in YAML does **not** read `HOST_TOKEN` from the
process running demonator — it sets `TOKEN` to the literal string
`${HOST_TOKEN}`.

To pull a value from the host environment, reference it in the command
itself (the shell expands it):

```yaml
steps:
  - text: "echo $HOME/$USER"   # expanded by sh, not by demonator
```

Anything you export from the shell that runs `demonator` is also inherited
by the spawned commands by default — `env:` only adds to (or overrides) that
inherited environment.

## Use cases

### Pinning a profile or region

```yaml
env:
  AWS_PROFILE: demo
  AWS_REGION: us-east-1
```

### Switching log verbosity for one step

```yaml
env:
  LOG_LEVEL: info

steps:
  - text: "./app start"
  - text: "./app diagnose"
    env:
      LOG_LEVEL: trace
```

### Disabling colors / pagers for clean recorded output

```yaml
env:
  NO_COLOR: "1"
  PAGER: cat
  GIT_PAGER: cat
```

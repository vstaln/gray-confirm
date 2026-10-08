# gray-confirm

Confirmation gate for dangerous `bash` commands — a gray sidecar plugin.
Port of pi's `permission-gate` + `confirm-destructive` + `timed-confirm`
extensions.

`tool/before` scans every `bash` command for dangerous patterns and, on a
match, asks via `host/ask` whether to run it. Empty, timed-out, or
unavailable answer channels deny — the gate fails closed.

## Patterns

Built-ins: `rm -rf`/`-r`/`--recursive`, `sudo`, `chmod`/`chown 777`, `dd`,
`mkfs*`, `shred`, `> /dev/…`, `kill -9 1`, `git push --force`/`-f`,
`git reset --hard`.

User extensions live in `~/.gray/confirm/patterns.json`:

```json
{"patterns": ["\\bdocker\\s+system\\s+prune\\b"]}
```

Invalid regexes are skipped (listed by `/confirm list`).

## Timed confirm

`~/.gray/confirm/config.json` ports pi's `timed-confirm`:

```json
{"timeout_secs": 30, "default": "deny"}
```

- `timeout_secs` — seconds to wait on the confirmation. `0` (default) means
  the host default (the sidecar's internal cap is 300s, under the host's
  330s outer TTL). `GRAY_CONFIRM_TIMEOUT_SECS` overrides this value.
- `default` — `"deny"` (default) or `"allow"`: the verdict when the ask
  times out or no answer channel exists.

## Commands

- `/confirm` — status (on/off, pattern count, timeout config)
- `/confirm list` — show active patterns
- `/confirm on` / `/confirm off` — toggle (state: `~/.gray/confirm/disabled`)

## Wire methods used

- `plugin/manifest`, `plugin/shutdown`
- `tool/before` (hook) — `allow`/`deny` verdicts
- `command/run` — `/confirm`
- `host/ask` (sidecar→host, capability `host.ask`, protocol 2.0)

## Install / consent

```sh
gray plugin install confirm
gray plugin capabilities confirm --all   # grants host.ask
```

Without the `host.ask` grant the gate still works: every dangerous command
denies with "no confirmation channel" (pi's headless behavior).

## Develop

```sh
cargo test
cargo build --release
gray account check
```

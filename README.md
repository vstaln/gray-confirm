<p align="center">
  <img src="assets/gray-logo.svg" alt="gray" width="96">
</p>
<h1 align="center">gray-confirm</h1>
<p align="center">A confirmation gate in front of dangerous bash commands — fail closed by default.</p>
<p align="center">
  <a href="https://github.com/vstaln/gray-confirm/blob/main/LICENSE"><img alt="MIT License" src="https://img.shields.io/badge/license-MIT-blue.svg"></a>
  <img alt="gray plugin" src="https://img.shields.io/badge/gray-plugin-7aa2f7.svg">
  <img alt="rust" src="https://img.shields.io/badge/built%20with-rust-orange.svg">
</p>

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

`~/.gray/confirm/config.json` controls the timeout and the default verdict:

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
denies with "no confirmation channel" — the gate stays fail-closed.

## Develop

```sh
cargo test
cargo build --release
gray account check
```

---
Part of the [gray](https://github.com/vstaln/gray) plugin ecosystem —
the open-source AI agent harness. <https://gray.alignment.id>

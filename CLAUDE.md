# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A Claude Code statusline plugin. It reads Claude Code's session JSON from stdin, calls Anthropic's OAuth usage API, and prints a two-line ANSI statusline showing model, project/branch, context %, tokens, and 5h/7d quota usage. Pure standard library — no dependencies, no build step, no package manager.

## Commands

Run the smoke tests (this is the only test suite; it doubles as CI on all three OSes):

```bash
python tests/smoke_test.py
```

Run a single check by importing and calling one function directly, e.g.:

```bash
python -c "import sys; sys.path.insert(0, 'tests'); from smoke_test import smoke_bar_toggle; smoke_bar_toggle()"
```

Manually exercise the statusline with fake input (mirrors what Claude Code pipes in):

```bash
echo '{"model":{"display_name":"Opus"},"context_window":{"used_percentage":25,"context_window_size":200000,"total_input_tokens":100,"total_output_tokens":50},"cost":{"total_cost_usd":0.1,"total_duration_ms":60000},"workspace":{"project_dir":"'"$PWD"'"}}' | python3 statusline.py
```

Test the shell/cmd launchers directly:

```bash
printf '' | bash statusline.sh
type nul | statusline.cmd    # Windows
```

Test the installer against a scratch target instead of the real `~/.claude`:

```bash
python install.py --source-dir . --install-dir /tmp/test-install --settings-path /tmp/test-settings.json
```

CI runs `python tests/smoke_test.py` on `ubuntu-latest`, `macos-latest`, and `windows-latest` via `.github/workflows/smoke-tests.yml` (see that matrix before assuming a change is cross-platform safe).

## Architecture

**`statusline.py`** is the entire runtime and is invoked fresh on every Claude Code statusline refresh (stdin in, two lines of ANSI out, process exits). Everything is top-level script code, not functions in a class — read it top to bottom:

1. Env vars (`CQB_*`) are read once at import time to decide which segments to show.
2. stdin is parsed as JSON; any parse failure or empty input prints `Claude` and exits 0 immediately — the statusline must never show an error to the user.
3. Session fields (model, context %, tokens, cost, project dir) are pulled defensively (`try`/`except KeyError, TypeError` around every field) since Claude Code's payload shape can vary.
4. Git branch is resolved by shelling out to `git rev-parse --abbrev-ref HEAD` against the project dir or cwd.
5. The OAuth token is read via `get_oauth_token()`: env var override → macOS Keychain (`security find-generic-password -s "Claude Code-credentials"`) → legacy `~/.claude/.credentials.json` file. This ordering matters — Keychain is the current source of truth on macOS.
6. Quota data comes from `https://api.anthropic.com/api/oauth/usage`, cached in `$TMPDIR/claude-sl-usage.json` for 5 minutes (`CACHE_TTL`). A lock file (`claude-sl-usage.lock`) prevents concurrent fetches across the many statusline processes Claude Code spawns; a fetch is kicked off in a background thread and the script blocks up to 8s at the very end (`_fetch_thread.join`) so the cache gets populated even though stale/first-run output still prints immediately with `--`.
7. Enterprise plans return `null` for `five_hour`/`seven_day` utilization but have `extra_usage` populated — this is detected via `is_enterprise = u5 is None and u7 is None` and switches the second line to a spend-limit display instead of the normal 5h/7d bars.
8. All numeric displays go through `used_pct_str()`, which conditionally inverts used%→remaining% (`CQB_REMAINING`) and conditionally renders a 5-block bar (`CQB_BAR`) — keep new percentage-based segments consistent with this helper rather than formatting inline.

**Launchers** (`statusline.sh`, `statusline.cmd`) are dumb dispatchers: they find a working Python (`python3`/`python`/`py -3` in that preference order on Unix; `py`/`python`/`python3` on Windows) and exec `statusline.py`, falling back to printing `Claude` if no Python is found. Keep these two launchers behaviorally identical when changing fallback logic — the smoke tests check both.

**`install.py`** is the actual installer logic (argparse-based, testable via `--source-dir`/`--install-dir`/`--settings-path` overrides). `install.sh`/`install.ps1` are thin bootstrap wrappers: they locate Python, then either delegate to a local `install.py` (audit-first / git-clone flow) or download the four runtime files from a pinned tag (`CLAUDE_USAGE_MONITOR_REF`, currently `v0.1.2`) via curl/wget and run the downloaded `install.py` (curl-pipe-bash quickstart flow). Both flows must stay behaviorally equivalent. The installer merges into an existing `settings.json` (preserving unrelated keys), backs up to `.bak` only if content actually changed, and runs `verify_install()` (expects literal stdout `Claude` on empty stdin) unless `--skip-verify`.

## Conventions

- No new dependencies — stdlib only, on Python 3.10+ (per README) / 3.11 (per CI).
- Any change to installer or launcher behavior needs a corresponding smoke test update (`tests/smoke_test.py`) — this is stated in `CONTRIBUTING.md` and enforced only by review, not tooling.
- The statusline must never raise or print a traceback to the user — preserve the defensive `try/except` + fallback-to-`"Claude"` pattern for any new field access or network call.
- Version pins for the curl-pipe-bash / irm installers live in the README quickstart commands and `install.sh`'s `REF` default — bump both together when cutting a release, alongside `CHANGELOG.md`.

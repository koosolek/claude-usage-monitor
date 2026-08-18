# Changelog

## Unreleased

### Changed
- Default to used % (growing bar) for 5h/7d quotas, matching the context gauge and Enterprise spend display. Set `CQB_REMAINING=1` to restore the draining remaining-% fuel gauge.
- All progress bars (context, 5h/7d, Enterprise spend) shrink from 5 blocks to 4 to save space.
- Once a quota hits 100% used, its bar and percentage are replaced by a `resets in <time>` countdown instead of a maxed-out, uninformative bar.

### Removed
- `CQB_RESET` and `CQB_DURATION` env vars and the session-duration segment - reset countdowns are now automatic (see above) rather than an always-on toggle, and session duration wasn't useful on a glanceable statusline.

### Fixed
- Context gauge and its smoke test/README description had drifted out of sync since the Enterprise support change (used% in code, remaining% in docs/tests) - both now consistently document/test used%.

## v0.1.2

### Changed
- Default to remaining % (fuel gauge) for all metrics - context, 5h, and 7d now all count down consistently. Set `CQB_REMAINING=0` to restore used % for quotas.

## v0.1.1

### Added
- Visual progress bar for 5h/7d quotas (on by default, disable with `CQB_BAR=0`)
- Clear `no token` message when OAuth credentials are missing instead of silent `--`

## v0.1.0

Initial release.

- 5h/7d quota tracking with color-coded percentages
- Context window usage gauge
- Token counts, reset countdowns, session duration
- One-command install for Windows, macOS, and Linux
- Configurable segments via environment variables
- `CQB_REMAINING` option to show remaining % instead of used %

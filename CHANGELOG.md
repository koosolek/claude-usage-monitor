# Changelog

## Unreleased

### Fixed
- Progress bars now use `■□` (U+25A0/U+25A1) instead of `▰▱` (U+25B0/U+25B1). Common coding fonts like JetBrains Mono lack the old glyphs, so terminals such as Ghostty fell back to another font and the bars rendered misaligned and mis-sized.

## v0.1.3

### Changed
- Default to used % (growing bar) for 5h/7d quotas, matching the context gauge and Enterprise spend display. Set `CQB_REMAINING=1` to restore the draining remaining-% fuel gauge.
- All progress bars (context, 5h/7d, Enterprise spend) shrink from 5 blocks to 4 to save space.
- Once a quota hits 100% used, its bar and percentage are replaced by a `resets in <time>` countdown instead of a maxed-out, uninformative bar.

### Removed
- `CQB_RESET` and `CQB_DURATION` env vars and the session-duration segment - reset countdowns are now automatic (see above) rather than an always-on toggle, and session duration wasn't useful on a glanceable statusline.

### Fixed
- Context gauge and its smoke test/README description had drifted out of sync since the Enterprise support change (used% in code, remaining% in docs/tests) - both now consistently document/test used%.

### Docs
- README, SECURITY.md, badges, and install links now point at this fork (`koosolek/claude-usage-monitor`) instead of upstream, since defaults have diverged.
- Documented the Enterprise spend-limit display (previously undocumented) with a dedicated screenshot.
- Refreshed every preset screenshot and the demo GIF to reflect the current used%/4-block/reset-on-exhaustion behavior; added a screenshot for the exhausted-quota state.

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

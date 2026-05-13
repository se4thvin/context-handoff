# Changelog

All notable changes to `context-handoff` are documented here. The project follows [Semantic Versioning](https://semver.org/).

## [0.1.4] — 2026-05-13

### Added
- `CHANGELOG.md` to track release history.
- `CONTRIBUTING.md` and `SECURITY.md` for project hygiene.
- `"license": "MIT"` field in `plugin.json` for tooling discoverability.
- README badges for version, license, and Claude Code compatibility.

### Changed
- Slugify rules in `writing-handoff` are now deterministic across sessions (explicit bash one-liner + 40-char cap + empty-result handling).
- Implementation history (`docs/specs/`, `docs/plans/`, `docs/tests/`) moved under `docs/development/` to declutter the top-level view.

### Fixed
- Absolute path containing real-name path component redacted from the verbatim baseline response in `docs/tests/baseline-writing-handoff.md`.

## [0.1.3] — 2026-05-12

### Fixed
- **Marketplace install bug:** top-level `name` in `marketplace.json` previously matched the plugin name (`context-handoff`), triggering Claude Code [issue #56043](https://github.com/anthropics/claude-code/issues/56043) which misclassifies the install as a "remote plugin with unrecognized source." Renamed the marketplace to `se4thvin-plugins`.

### Changed
- Slash command `/recap` renamed to `/handoff-resume` for clarity.

## [0.1.2] — 2026-05-12

### Fixed
- Switched marketplace `source` from the unsupported `"."` shorthand to the verified-working `{ "source": "url", "url": "...", "sha": "..." }` form. Pinning to a commit SHA so installs are reproducible.

### Changed
- Slash command `/resume` renamed to `/recap` to avoid collision with Claude Code's built-in `/resume`.

## [0.1.1] — 2026-05-12

### Added
- `.claude-plugin/marketplace.json` so `/plugin marketplace add se4thvin/context-handoff` actually resolves (this was missing in v0.1.0 — the install would 404).

### Changed
- `writing-handoff` skill's "Confirm to the user" step now mandates the exact two-line `✓ Wrote ... / Captured: ...` format that the README shows.
- README's "auto-load if only one exists" softened to "auto-summarize (then asks before continuing)" to match the skill's confirm-before-acting discipline.

## [0.1.0] — 2026-05-12

### Added
- Initial release.
- `writing-handoff` skill — fires on explicit `/handoff`, "write a handoff" / "save state" phrasing, or context-exhaustion signals.
- `resuming-from-handoff` skill — fires on `/resume` or "pick up where we left off"; shows a numbered picker for multiple handoffs and confirms before acting.
- Slash commands `/handoff [label]` and `/resume [n]`.
- TDD-for-skills validation: RED-phase baselines + GREEN-phase verification captured in `docs/tests/`.

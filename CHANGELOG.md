# Changelog

## 2.0.0 — 2026-11

Rewrite.

### BREAKING
- The role is renamed `pastir.td_agent` → `pastir.fluentd` (repository `ansible-role-fluentd`).
- Default package is **fluent-package** LTS 6 (`fluentd_package`). td-agent v4 is still supported.
- `td_agent_plugin` / `__td_agent_plugin` → `fluentd_gems` (`[{name, version}]`).
- The bundled `out_gelf.rb` moved to `files/` and is installed only with `fluentd_gelf_plugin: true`.

### Fixed
- Installation ran `curl ... | sh` on every run. There was no td-agent v4 script for 24.04, and nothing was pinned.
- Gems were reinstalled on every run.
- The configuration was not managed at all (`tasks/config` was empty).

### Added
- deb822 repositories, version pin and hold with an upgrade guard.
- Config files with `fluentd --dry-run` and rollback, graceful reload.
- Molecule tests for 24.04 and 26.04.

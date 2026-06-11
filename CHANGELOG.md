# Changelog

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] — 2026-06-11

### Added
- Initial public release. A **Claude Code / Codex skill** that installs and remotely
  operates the [`xray-relay`](https://github.com/superchaospc/xray-relay) one-click
  **VLESS + REALITY** relay script (TCP + XTLS-Vision) on a VPS over SSH, driving its
  16-item interactive menu non-interactively (stdin-feed + EOF contract).
- `SKILL.md` — when to trigger + the install/drive flow.
- `references/menu-actions.md` — exact stdin sequence for every menu item.
- `install.sh` — self-contained cross-tool installer (symlinks the skill into Codex's
  `~/.codex/skills` / `~/.agents/skills`).
- Bilingual (English / 中文) README, MIT LICENSE.

Sibling of [`xray-xhttp-relay-deploy`](https://github.com/superchaospc/xray-xhttp-relay-deploy)
(the XHTTP variant); they share the identical 16-item menu.

[1.0.0]: https://github.com/superchaospc/xray-relay-deploy/releases/tag/v1.0.0

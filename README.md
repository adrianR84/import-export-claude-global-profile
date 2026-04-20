# Claude Profile Sync

**Backup, restore, and diff your Claude Code global settings** — fully local or with a GitHub remote.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue?style=flat-square)](LICENSE)
[![Bash 3.2+](https://img.shields.io/badge/Bash-3.2+-3c873a?style=flat-square)](https://www.gnu.org/software/bash/)
[![Node.js](https://img.shields.io/badge/Node.js-20+-3c873a?style=flat-square)](https://nodejs.org)

---

## Features

- **Diff** — preview differences before making any changes
- **Export** — back up `~/.claude/` to a local folder or GitHub repo
- **Import** — restore from a backup onto the same machine or a new one
- **Merge or clean sync** — choose between safe merge or exact match
- **Cross-platform** — plugin paths are sanitized to relative form; portable across Windows, macOS, Linux
- **GitHub optional** — works fully offline; enable it to sync across machines

---

## Prerequisites

- [Claude Code](https://claude.com/claude-code) installed
- `bash` (Git Bash on Windows, terminal bash on macOS/Linux)
- `node` (for plugin path sanitization)

---

## Installation

```bash
git clone https://github.com/adrianR84/import-export-claude-global-profile \
  ~/.claude/skills/import-export-claude-global-profile
```

Restart Claude Code — it auto-discovers skills by scanning for `SKILL.md` files.

---

## Configuration

Edit `~/.claude/skills/import-export-claude-global-profile/config.yml`:

```yaml
# ─── Backup ───────────────────────────────────────────────────────────────────

# Local backup folder (default: ~/.claude-profile)
backup_folder: ~/.claude-profile

# GitHub remote — leave empty to disable remote sync (local-only mode)
github_repo: https://github.com/yourusername/your-repo-name

# ─── What to sync ──────────────────────────────────────────────────────────────

# Folders to sync (space-separated, relative to ~/.claude/)
folders: skills agents commands hooks

# Plugin files to sync (space-separated, from plugins/)
plugin_files: installed_plugins.json known_marketplaces.json

# Individual files to sync (space-separated)
files: settings.json AGENTS.md CLAUDE.md
```

| Key | Default | Description |
|-----|---------|-------------|
| `backup_folder` | `~/.claude-profile` | Local backup folder |
| `github_repo` | _(none)_ | GitHub repo URL. Empty = local-only |
| `folders` | `skills agents commands hooks` | Folders to sync |
| `plugin_files` | `installed_plugins.json known_marketplaces.json` | Plugin JSON files |
| `files` | `settings.json AGENTS.md CLAUDE.md` | Individual files |

> [!TIP]
> The skill activates when you say things like "export my claude profile", "backup claude settings", "restore claude from backup", or "compare claude settings".

---

## Operations

### Diff — preview changes

See exactly what differs between your live config and your backup before making any changes.

```bash
bash ~/.claude/skills/import-export-claude-global-profile/scripts/diff.sh
```

Output symbols:
- `✓` — identical
- `≠` — differs
- `→` — only in `~/.claude/` (not yet backed up)
- `←` — only in backup (not yet restored)

---

### Export — back up

Save your current `~/.claude/` settings to the backup folder.

```bash
bash ~/.claude/skills/import-export-claude-global-profile/scripts/export.sh [merge|clean]
```

**Sync modes:**

| Mode | Behavior |
|------|----------|
| `merge` (default) | Source items added/updated in backup. Backup-only items preserved. Safe, no data loss. |
| `clean` | Backup made to exactly match source. Items not in source are deleted. |

If `github_repo` is configured, export also commits and pushes to the remote.

---

### Import — restore

Restore settings from the backup folder to `~/.claude/`.

```bash
bash ~/.claude/skills/import-export-claude-global-profile/scripts/import.sh [merge|clean]
```

Sync modes are the same as export. Merge is recommended.

> [!IMPORTANT]
> After importing, run `/reload-plugins` in Claude Code to load restored plugins.

If `github_repo` is configured, import pulls from the remote first.

---

## Plugin path sanitization

Plugin files (`installed_plugins.json`, `known_marketplaces.json`) store absolute filesystem paths to installed plugins — these are machine-specific and would break if restored on another OS.

**Export** converts absolute paths to relative, OS-neutral form:
```
C:\Users\you\.claude\plugins\cache\foo\1.0.0  →  plugins/cache/foo/1.0.0
```

**Import** converts them back to absolute paths for the current machine:
```
plugins/cache/foo/1.0.0  →  /home/you/.claude/plugins/cache/foo/1.0.0
```

This means your backup works correctly regardless of which machine you restore to.

---

## Skill structure

```
import-export-claude-global-profile/
├── SKILL.md              # Skill manifest (required by Claude Code)
├── README.md             # This file
├── config.yml            # User settings
└── scripts/
    ├── _config.sh        # YAML configuration loader (pure bash)
    ├── diff.sh           # Preview differences
    ├── export.sh         # Backup to folder / GitHub
    ├── import.sh         # Restore from backup
    └── sanitize-json.js  # Cross-platform plugin path sanitizer
```

---

## File safety

- **Merge mode** — never deletes items. Only adds and updates from source. Recommended for everyday use.
- **Clean mode** — removes items not present in source. Destructive; use with caution.

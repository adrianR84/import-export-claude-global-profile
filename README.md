# Import / Export Claude Code Global Profile

Backup, restore, and compare your Claude Code global settings — fully local or with a GitHub remote.

---

## Installation

### Requirements

- [Claude Code](https://claude.com/claude-code) installed
- `bash` available (Git Bash on Windows, terminal bash on macOS/Linux)
- `node` available (for plugin path sanitization)

### Steps

**1. Copy the skill to your Claude Code skills folder:**

```bash
# Clone or copy the skill directory into ~/.claude/skills/
git clone <your-repo-url> ~/.claude/skills/import-export-claude-global-profile
```

**2. (Optional) Configure the skill:**

Edit `~/.claude/skills/import-export-claude-global-profile/config.yml` to set your backup folder and optionally a GitHub remote.

**3. Restart Claude Code** — it auto-discovers skills on startup.

That's it. The skill activates when you say something like "export my claude profile", "backup claude settings", "restore claude from backup", or "compare my claude settings".

### Skill structure

```
import-export-claude-global-profile/
├── SKILL.md          # Skill manifest (required by Claude Code)
├── README.md         # This file
├── config.yml        # User-facing settings
└── scripts/
    ├── _config.sh    # Shared configuration loader
    ├── diff.sh       # Preview differences
    ├── export.sh     # Backup to folder / GitHub
    ├── import.sh     # Restore from backup
    └── sanitize-json.js  # Plugin path sanitization
```

Claude Code auto-discovers skills in `~/.claude/skills/` by scanning for `SKILL.md` files. No registration step needed.

---

## What it syncs

| Type | Items |
|------|-------|
| Folders | `skills/`, `agents/`, `commands/`, `hooks/` |
| Individual files | `settings.json`, `AGENTS.md`, `CLAUDE.md` |
| Plugin files | `plugins/installed_plugins.json`, `plugins/known_marketplaces.json` |

All paths are relative to `~/.claude/`.

---

## Quick start

### 1. First time? Review the configuration

All settings live in:

```
~/.claude/skills/import-export-claude-global-profile/config.yml
```

Open it to confirm or change:
- **Backup folder** — where your backups are stored locally
- **GitHub repo** — optional; leave empty for local-only mode
- **What to sync** — folders, files, and plugin files

### 2. Choose an operation

| Operation | When to use |
|-----------|-------------|
| **Diff** | Preview differences between `~/.claude/` and your backup |
| **Export** | Save your current settings to the backup folder |
| **Import** | Restore settings from the backup folder to `~/.claude/` |

---

## Operations in detail

### Diff (preview changes)

See exactly what differs between your live config and your backup — before making any changes.

```
bash ~/.claude/skills/import-export-claude-global-profile/scripts/diff.sh
```

Output shows:
- `✓` identical files
- `≠` files that differ
- `→` items only in `~/.claude/` (not yet backed up)
- `←` items only in backup (not yet restored)

---

### Export (backup)

Save your current `~/.claude/` settings to the backup folder. Optionally push to GitHub.

```
bash ~/.claude/skills/import-export-claude-global-profile/scripts/export.sh [merge|clean]
```

**Sync modes:**
- `merge` (default) — source items added/updated in backup. Backup-only items are preserved. Safe, no data loss.
- `clean` — backup made to exactly match source. Items not in source are deleted from backup. Use with caution.

---

### Import (restore)

Restore settings from the backup folder to `~/.claude/`. Optionally pull from GitHub first.

```
bash ~/.claude/skills/import-export-claude-global-profile/scripts/import.sh [merge|clean]
```

Existing `~/.claude/` files are renamed to `.backup` before being overwritten.

**Sync modes:** same as export — `merge` (default) or `clean`.

**After import:** run `/reload-plugins` in Claude Code to load restored plugins.

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

# Folders to sync (space-separated)
folders: skills agents commands hooks

# Plugin files to sync (space-separated, from plugins/)
plugin_files: installed_plugins.json known_marketplaces.json

# Individual files to sync (space-separated)
files: settings.json AGENTS.md CLAUDE.md
```

| Key | Default | Description |
|-----|---------|-------------|
| `backup_folder` | `~/.claude-profile` | Local backup folder path |
| `github_repo` | _(none)_ | GitHub repository URL. Empty = local-only mode |
| `folders` | `skills agents commands hooks` | Space-separated folder list to sync |
| `plugin_files` | `installed_plugins.json known_marketplaces.json` | Space-separated plugin file list |
| `files` | `settings.json AGENTS.md CLAUDE.md` | Space-separated individual file list |

---

## GitHub sync

If `github_repo` is set in `config.yml`:

- **Export** commits and pushes to the configured repository
- **Import** pulls from the repository before restoring

If `github_repo` is empty, all operations are fully local.

---

## Plugin path sanitization

Plugin files (`installed_plugins.json`, `known_marketplaces.json`) store absolute paths to local plugins. When exporting, paths are sanitized to relative form so the backup is portable. When importing, they are restored to absolute paths for the current machine.

---

## File safety

- Import overwrites are preceded by automatic `.backup` suffix renames
- Merge mode never deletes items — only adds/updates from source
- Clean mode is the only destructive option — it removes items not present in source

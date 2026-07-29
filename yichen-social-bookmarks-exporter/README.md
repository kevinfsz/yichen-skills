# yichen-social-bookmarks-exporter

Export the currently accessible Xiaohongshu favorites, Douyin favorites, and X bookmarks into local URL files, then validate counts, duplicates, and URL shape.

## Privacy boundary

- Private collections are read only after explicit authorization for the current task.
- The workflow does not like, favorite, comment, follow, publish, or modify account data.
- It does not print or export cookies, browser storage, passwords, or token databases.
- Xiaohongshu URLs may contain `xsec_token`; those URLs stay only in the user's requested local export file.
- No real cookies, tokens, account identifiers, private URLs, export results, or personal filesystem paths are included in this repository.

## Routes

- Xiaohongshu: reuse the user's authenticated Chrome tab and scroll the Favorites → Notes panel.
- Douyin: reuse the user's authenticated Chrome tab and scroll the Favorites → Videos panel.
- X: call a separately installed Field Theory `ft` CLI build whose version contains `graphql-only`.

The X route is an optional external dependency. This repository does not include Field Theory, private query IDs, cookies, or credentials.

## Files

- `SKILL.md`: workflow and safety rules
- `references/chrome-collections.md`: Chrome navigation and bottom-detection details
- `scripts/chrome_collectors.mjs`: read-only Xiaohongshu and Douyin collectors
- `scripts/export_x_links.py`: URL-only export from the local Field Theory index
- `scripts/validate_link_file.py`: privacy-safe URL-file validation

## Install

Copy `yichen-social-bookmarks-exporter/` into a skill directory loaded by Claude Code or Codex, keeping the directory name unchanged.

Common locations include:

- `~/.agents/skills/yichen-social-bookmarks-exporter/`
- `~/.claude/skills/yichen-social-bookmarks-exporter/`
- `~/.codex/skills/yichen-social-bookmarks-exporter/`

Restart the agent session after installation.

## Requirements

- An agent environment that provides `chrome:control-chrome` for Xiaohongshu and Douyin
- Node.js 18+ for `chrome_collectors.mjs`
- Python 3.9+ for the X exporter and link validator
- Optional: a compatible Field Theory `ft` CLI build reporting `graphql-only`

## Validation examples

```bash
python3 "<SKILL_DIR>/scripts/validate_link_file.py" \
  --platform douyin "/absolute/output/douyin.txt"

python3 "<SKILL_DIR>/scripts/export_x_links.py" \
  --output "/absolute/output/x-bookmarks.txt"
```

The X command refuses to overwrite an existing file unless `--overwrite` is passed. The Skill itself requires explicit user approval before any overwrite.

## License

See the repository-level license.

---
name: sync-skill-to-claude-desktop
description: Upload Claude Code skills to claude.ai for use in Claude Desktop. Use when the user says "sync skill to desktop", "upload skill to claude.ai", "sync all skills to desktop", or wants to make their Claude Code skills available in Claude Desktop.
---

# Sync skill to Claude Desktop

Upload Claude Code skills from `~/.claude/skills/` to claude.ai, making them available in Claude Desktop via cloud sync.

## Commands

- `/sync-skill-to-desktop <skill-name>` - Upload a single skill
- `/sync-all-skills-to-desktop` - Upload all skills from `~/.claude/skills/`

## Important: Frontmatter compatibility

Claude.ai skills only accept these frontmatter keys:
- `name` (required)
- `description` (required)
- `license` (optional)
- `allowed-tools` (optional)
- `compatibility` (optional)
- `metadata` (optional)

**Common issue:** Claude Code plugin skills often use `version` in frontmatter, which claude.ai rejects. This skill automatically removes invalid keys before creating the zip.

## Workflow

### 1. Validate skills exist

For each skill to upload, check it exists:
```bash
ls ~/.claude/skills/<skill-name>/SKILL.md 2>/dev/null || ls ~/.claude/skills/<skill-name>/skill.md
```

List available skills if needed:
```bash
ls -d ~/.claude/skills/*/ | xargs -I {} basename {}
```

### 2. Check and fix frontmatter

For each skill, check for invalid frontmatter keys:
```bash
head -20 ~/.claude/skills/<skill-name>/SKILL.md
```

If `version` or other invalid keys exist, remove them before creating the zip.

### 3. Create output folder

```bash
mkdir -p /tmp/claude-skills-upload
rm -f /tmp/claude-skills-upload/*.zip
```

### 4. Create zip files

For each skill:
```bash
cd ~/.claude/skills && zip -r /tmp/claude-skills-upload/<skill-name>.zip <skill-name> \
  -x "*.git*" \
  -x "*node_modules*" \
  -x "*.env" \
  -x "*.local.md" \
  -x "*.DS_Store" \
  -x "*__pycache__*"
```

### 5. Open folder in Finder

```bash
open /tmp/claude-skills-upload
```

### 6. Provide instructions

Tell the user:

```
I've created zip files for your skills in Finder.

To upload them to claude.ai:
1. Open https://claude.ai/settings/capabilities in Chrome
2. Scroll to the Skills section
3. Click "+ Add" → "Upload a skill"
4. Drag each zip file from Finder to the upload area (or click to browse)

Skills ready to upload:
- chief-of-staff.zip
- summarise-granola.zip
- etc.

After uploading, restart Claude Desktop to sync the new skills.
```

## Sync-all workflow

### 1. Find all valid skills

```bash
for dir in ~/.claude/skills/*/; do
  if [ -f "${dir}SKILL.md" ] || [ -f "${dir}skill.md" ]; then
    basename "$dir"
  fi
done
```

### 2. Process each skill

For each skill found:
1. Check frontmatter for invalid keys
2. Fix if needed (remove `version`, etc.)
3. Create zip in `/tmp/claude-skills-upload/`

### 3. Open folder and provide instructions

Same as single skill workflow.

## Edge cases

### Invalid frontmatter

If a skill has invalid frontmatter keys:
1. Report which key is invalid
2. Offer to fix it (remove the invalid key)
3. Only create zip after fixing

### Skill file naming

Skills may use either `SKILL.md` or `skill.md` - check for both.

## Important notes

- Skills are uploaded as zip files containing the entire skill directory
- The zip must contain a `SKILL.md` file
- Excluded from zip: `.git`, `node_modules`, `.env`, `.local.md` files, `.DS_Store`
- Only frontmatter keys `name`, `description`, `license`, `allowed-tools`, `compatibility`, `metadata` are allowed
- User may need to restart Claude Desktop to see new skills after uploading

## Update check

This is a shared skill. Before executing, check `~/.claude/skills/.update-config.json`.
If `auto_check_enabled` is true and `last_checked_timestamp` is older than `check_frequency_days`,
mention: "It's been a while since skill updates were checked. Run `/update-skills` to see available updates."
Do NOT perform network operations - just check the local timestamp.

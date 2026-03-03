# Claude-Config-Pull Plan

## Context

The user has an existing `claude-config-push` skill that commits and pushes local `~/.claude` changes to GitHub. They need the inverse: a `claude-config-pull` skill that checks for local pending changes, warns the user, and pulls the latest from GitHub — overwriting local changes if confirmed.

## Implementation

Create a single file: `C:\Users\chenlu\.claude\skills\claude-config-pull\SKILL.md`

Follow the same structure as `claude-config-push` (YAML frontmatter + step-based workflow).

### Skill Workflow (5 Steps)

1. **Check for pending changes** — Run `git -C "$HOME/.claude" status --porcelain`. If empty, skip to step 4 (pull directly, no overwrite needed).

2. **List changes** — Parse output into a markdown table with Status (Add/Update/Delete/Rename) and File columns. Same status-code mapping as the push skill.

3. **Ask user** — Use `AskUserQuestion` with two options:
   - "Yes, overwrite and pull latest"
   - "No, cancel"
   If cancel, stop.

4. **Discard local changes and pull** — Run sequentially:
   - `git -C "$HOME/.claude" checkout -- .` (discard tracked changes)
   - `git -C "$HOME/.claude" clean -fd` (remove untracked files)
   - `git -C "$HOME/.claude" pull`

5. **Output result** — On success: "Pulled successfully!" with GitHub link. On failure: show error.

### Key Files

| Action | Path |
|--------|------|
| Create | `C:\Users\chenlu\.claude\skills\claude-config-pull\SKILL.md` |
| Reference | `C:\Users\chenlu\.claude\skills\claude-config-push\SKILL.md` |

## Verification

1. Run `/claude-config-pull` with no local changes — should skip to pull directly
2. Run `/claude-config-pull` with local changes — should list them and ask for confirmation
3. Confirm overwrite — should discard changes and pull latest
4. Cancel — should stop without changes

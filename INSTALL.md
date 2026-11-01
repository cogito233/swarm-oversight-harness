# INSTALL (for agents)

You are a coding agent installing Swarm Oversight Harness for your user. Follow these steps exactly. Ask before overwriting anything.

## 1 · Get the repo

```bash
REPO="$HOME/.swarm-oversight-harness"
if [ -d "$REPO/.git" ]; then git -C "$REPO" pull --ff-only; else git clone https://github.com/cogito233/swarm-oversight-harness.git "$REPO"; fi
```

## 2 · Link the skills

Pick the skills directory for the agent you are running in:

| Agent | Skills directory |
|---|---|
| Claude Code (user-level) | `~/.claude/skills/` |
| Claude Code (this project only) | `<project>/.claude/skills/` |
| Codex | `~/.codex/skills/` |
| DeepSeek Harness (untested) | `<project>/.agents/skills/` |
| Other agents with Agent Skills support | that agent's skills directory |

```bash
SKILLS_DIR="$HOME/.claude/skills"   # change per the table
mkdir -p "$SKILLS_DIR"
for s in "$REPO"/skills/*/; do
  name=$(basename "$s")
  if [ -e "$SKILLS_DIR/$name" ] && [ ! -L "$SKILLS_DIR/$name" ]; then
    echo "SKIP $name: a non-symlink already exists — ask the user"; continue
  fi
  ln -sfn "$s" "$SKILLS_DIR/$name"
done
ls -l "$SKILLS_DIR" | grep swarm-
```

If your agent does not support skills, add this line to the project's `AGENTS.md` (or `CLAUDE.md`) instead:
`For project workflow, follow ~/.swarm-oversight-harness/skills/*/SKILL.md (setup, loop, accept).`

## 3 · Verify

List the five installed skills (`swarm-setup`, `swarm-loop`, `swarm-accept`, `swarm-complexity-audit`, `swarm-paper`), say which directory they were linked into, and tell the user that they may need to restart the agent session before the skills load.

## 4 · Offer the next step

Ask the user whether they want to run `swarm-setup` on the current repo now. Do not run it unless they say yes.

## Update / uninstall

- Update: `git -C ~/.swarm-oversight-harness pull --ff-only` (the symlinks pick up the new version).
- Uninstall: remove the `swarm-*` symlinks from the skills directory, then delete `~/.swarm-oversight-harness`.
- Pin a version per project (optional): add the repo as a git submodule and record `built_against: swarm-oversight-harness v<X.Y>` in CHARTER (MANUAL §9).

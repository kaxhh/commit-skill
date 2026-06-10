# Claude Code Commit Skill

[中文](README.md)

A Claude Code slash command skill for interactive Git commit creation.

## Features

- Automatic repository detection (supports multi-repo workspaces)
- **Interactive file selection**: checkbox-based multi-select for files, with directory-level bulk selection
- **Branch name as scope**: commit messages follow `type(branch): description` format, auto-detecting the current branch
- Strict confirmation flow: select files → review content → confirm commit
- No Co-Authored-By or other author trailer lines added

## Installation

Copy `SKILL.md` into your Claude Code commands directory:

```bash
mkdir -p ~/.claude/commands/commit
cp SKILL.md ~/.claude/commands/commit/
```

After installation, invoke `/commit` in Claude Code.

## Usage

```
/commit                          # current repo
/commit oneos-multi-os           # specify repo
/commit oneos-multi-os fix: bug  # specify repo + commit message
```

## Commit Message Format

Auto-generates conventional commit format with the current branch as scope:

```
feat(branch): add new feature
fix(branch): fix a bug
docs(branch): update documentation
refactor(branch): restructure code
test(branch): add tests
chore(branch): maintenance task
```

## Configuration

No additional configuration needed. The skill uses `AskUserQuestion` for interactive file selection.

## License

MIT
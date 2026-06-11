# Claude Code Commit Skill

[English](README.md) | [简体中文](README.zh-CN.md)

A Claude Code and Codex slash command skill for interactive Git commit creation. In environments without a multi-select UI, it falls back to numbered text selection.

## Features

- Automatic repository detection (supports multi-repo workspaces)
- **File selection**: prefers checkbox-based multi-select, falls back to numbered text selection, with directory-level bulk selection
- **Branch name as scope**: commit messages follow `type(branch): description` format, auto-detecting the current branch
- Strict confirmation flow: select files → review content → confirm commit
- No Co-Authored-By or other author trailer lines added

## Repository Layout

```text
commit-skill/
├── README.md
├── README.zh-CN.md
├── LICENSE
├── .gitignore
└── skills/
    └── commit/
        └── SKILL.md
```

## Installation

Copy the skill directory to your agents skills path and create a symlink in Claude Code's commands directory:

```bash
mkdir -p ~/.agents/skills
cp -a skills/commit ~/.agents/skills/

# Slash invocation (commands/ = slash + auto-trigger)
ln -s ~/.agents/skills/commit ~/.claude/commands/commit
```

Or install directly into the commands directory:

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

## Requirements

- Claude Code with `AskUserQuestion` support for interactive file selection. In Codex or other environments without multi-select UI, falls back to numbered text selection.
- No external dependencies.

## License

MIT
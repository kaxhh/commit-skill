# Claude Code Commit Skill

[English](README.en.md)

一个 Claude Code 的 slash command skill，用于交互式创建 Git commit；在 Codex 等没有多选框工具的环境中，会退化为编号文本选择。

## 功能

- 自动识别目标仓库（支持多仓库工作区）
- **文件选择**：优先使用多选框，工具不可用时使用编号文本选择，支持按目录批量选择
- **分支名作为 scope**：commit message 格式为 `type(branch): description`，自动获取当前分支名
- 严格的确认流程：选文件 → 确认内容 → 确认提交
- 不添加 Co-Authored-By 等无关信息

## 安装

将 `SKILL.md` 复制到 Claude Code 的 commands 目录：

```bash
mkdir -p ~/.claude/commands/commit
cp SKILL.md ~/.claude/commands/commit/
```

安装后，在 Claude Code 中使用 `/commit` 即可触发。

## 使用方式

```
/commit                          # 当前仓库
/commit oneos-multi-os           # 指定仓库
/commit oneos-multi-os fix: bug  # 指定仓库 + commit message
```

## Commit Message 格式

自动生成 conventional commit 格式，scope 为当前分支名：

```
feat(branch): add new feature
fix(branch): fix a bug
docs(branch): update documentation
refactor(branch): restructure code
test(branch): add tests
chore(branch): maintenance task
```

## 配置

无需额外配置。Skill 会优先使用 `AskUserQuestion` 提供交互式文件选择；如果当前环境不支持该工具，则列出编号选项并等待用户回复数字、路径或 `all`。

## 许可证

MIT

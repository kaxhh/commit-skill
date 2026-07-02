---
name: commit
description: Use when the user invokes /commit or asks to create a Git commit after coding. Select the target repository, show changed files for user selection, generate or validate a conventional commit message of 30 words or fewer, ask for explicit confirmation, then create exactly one commit containing only the selected files and push it to the configured remote. Also use for Chinese requests such as 提交代码, 写 commit, 生成提交, 提交并推送, or 选择文件提交.
allowed-tools: Bash(git:*), Bash(pwd), Bash(find:*), Bash(test:*), Bash(ls:*), AskUserQuestion
argument-hint: "[repo-path] [commit-message]"
---

# Commit Selected Changes

Create one Git commit only after repository resolution, file selection, review, and explicit user confirmation.

## Repository Resolution

Resolve the target repository conservatively:

1. If the first argument is an existing directory or path inside a Git work tree, use that repository.
   Examples: `/commit oneos-multi-os`, `/commit ./oneos-multi-bsp fix: add aio tests`.
2. If no repository path is supplied, infer from recent conversation only when exactly one repository is clear.
3. If the current working directory is inside a Git work tree, use that repository.
4. If the current directory is a parent directory, inspect immediate child directories for Git repositories with changes.
5. If more than one repository has changes and the target is ambiguous, list the candidate repositories and ask the user to specify one.
6. Never commit from a parent directory that is not itself a Git work tree.

After resolving the repository, use `git -C <repo> ...` for every Git operation.

## Remote Target

After resolving the repository, determine the push target:

1. Run `git -C <repo> rev-parse --abbrev-ref --symbolic-full-name @{u}` to get the upstream tracking branch.
2. If an upstream exists (e.g., `origin/main`), record it as the push target.
3. If no upstream is configured, record `origin/HEAD` as the intended remote and note that `git push -u origin <branch>` will be used to set upstream on first push.
4. Run `git -C <repo> remote` to verify at least one remote exists. If no remote is configured, note that push will be skipped and inform the user.

Never force-push. If the push is rejected (non-fast-forward, diverged branches), stop and report the rejection without attempting `--force` or `--force-with-lease`.

## File Selection

Before staging or committing:

1. Run `git -C <repo> status --short`.
2. If there are no changes, say so and stop.
3. Prefer an interactive multi-select tool when the runtime provides one, such as AskUserQuestion. If no such tool is available, use the text fallback below instead of stopping.

Interactive selection format:
- Use multiSelect: true so the user can pick multiple items.
- Each option's label should be the file path or directory path, description should note the change type (modified, untracked, deleted, etc.) and the number of files if it's a directory group.
- If files are already staged, note that in the description.
- If fewer than 4 items total, list individual files only. If 4 or more, group by directory where possible, but still offer individual files as separate options if they are important changes.
- The user can always type custom paths via "Other".

Text fallback when interactive selection is unavailable:
- Print a numbered list of changed file and directory options.
- Ask the user to reply with numbers, exact paths, directory labels, or `all`.
- Treat directory selections as all changed files under that directory.
- Echo the resolved file list back to the user and ask for explicit confirmation before staging.
- If the user's reply is ambiguous, ask a short clarification question instead of guessing.

Do not continue until at least one file is selected.

Commit granularity is whole-file only. If the user asks for partial hunks, stop and say this command currently supports file-level selection only.

For renamed or copied files, treat the old and new paths as one selectable item and stage both paths.

## Commit Message

Every commit message must be 30 words or fewer, counting whitespace-separated tokens.

If the user supplies a commit message after the repository path, use it as the proposed message exactly. If it exceeds 30 words, stop and ask for a shorter message.

If no message is supplied, inspect only the selected files and propose one conventional commit subject:

- `feat(scope):` for new features
- `fix(scope):` for bug fixes
- `docs(scope):` for documentation changes
- `refactor(scope):` for refactoring
- `test(scope):` for tests
- `chore(scope):` for maintenance

The `scope` MUST be the current Git branch name (obtained via `git -C <repo> branch --show-current`). For example, if the branch is `mnt_ns` and the change is a bug fix, the message should be `fix(mnt_ns): defer m_umount null check to ns_cnt zero branch with rollback`.

Keep the message concise and specific. Prefer 15 words or fewer.

## Required Review

Before asking for final confirmation, show:

- Target repository path
- Current branch
- Remote push target (upstream tracking branch or `origin/<branch>` if no upstream)
- `git status --short`
- `git diff --stat HEAD` if `HEAD` exists, otherwise the closest useful diff stat
- Selected files
- Proposed commit message

Inspect enough of the selected diff to ensure the message matches the changes.

If selected files include generated binaries, build outputs, archives, rootfs images, vendored dependencies, or unrelated files, call that out explicitly and ask whether to keep them selected.

## Confirmation Gate

Do not run `git add`, `git restore --staged`, `git commit`, or `git push` until the user has selected files and explicitly confirms.

Ask a direct confirmation question that includes the target repository, selected files, proposed commit message, and the remote push target. The confirmation must clearly state that committing will also push to the remote.

Proceed only when the user replies with a clear affirmative such as `yes`, `y`, `确认`, `可以`, or `提交`.

## Commit and Push Procedure

After confirmation:

1. If anything is staged, run `git -C <repo> restore --staged -- .` to clear the index without changing working-tree files.
2. Stage only selected files with `git -C <repo> add -A -- <selected-paths>`.
3. For renamed or copied selections, include both old and new paths in the pathspec.
4. Verify the staged set with `git -C <repo> diff --cached --name-status`.
5. If the staged set contains anything not selected, stop and report the mismatch.
6. Create one commit with `git -C <repo> commit -m "<message>"`. Do NOT add any Co-Authored-By or author trailer lines.
7. Show the resulting commit hash.
8. Push to the remote:
   - If an upstream tracking branch exists, run `git -C <repo> push`.
   - If no upstream is configured, run `git -C <repo> push -u origin HEAD` to set upstream on first push.
   - Never use `--force` or `--force-with-lease`.
   - If the push is rejected, stop and report the rejection message without retrying.
9. Show the push result and final `git -C <repo> status --short`.

Never use destructive cleanup commands such as `git reset --hard`, `git checkout --`, or `git clean`.

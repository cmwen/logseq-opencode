---
description: Stage, commit, and push current changes using generated
  Conventional Commit message
agent: build
model: github-copilot/gpt-5-mini
---

Generate a concise, Conventional Commits-style git commit message, stage
the relevant changes, create the commit, and push to the remote branch.

Use the following git commands for status and diffs (for context):

Current git status:
!`git status --porcelain --untracked-files=normal`

Recent git diff (staged if any, otherwise unstaged):
!`git diff --staged --name-only || git diff --name-only`

Full git diff for context (limit large output to ensure prompt fits):
!`git --no-pager diff --staged || git --no-pager diff`

Procedure for the agent:
- Generate a Conventional Commit message (type/scope; title <=72 chars).
- Provide a short 1-line title and a 1-3 sentence body explaining why.
- List key files changed (max 8).
- Stage the intended files using `git add`.
- Create the commit using `git commit -m "<title>" -m "<body>"`.
- Push the commit to the current branch's upstream using `git push`.

If you cannot execute git commands in this environment, output the
exact shell commands the user should run instead, formatted in three
sections separated by '---':

1) COMMIT_MESSAGE: the generated commit message (title and body).
2) FILES_CHANGED: short bullet list of key files changed (max 8).
3) GIT_COMMANDS: exact shell commands to run to commit and push.

Do not include any other commentary.

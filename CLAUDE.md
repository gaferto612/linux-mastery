# Claude working rules for this repo

These rules apply to every Claude Code session on this repository.

## Git workflow

- **Always open a pull request** for every branch with work, even small fixes. Do not stop at "pushed to branch."
- **Always merge the PR into `main`** once it's opened, unless the user explicitly says otherwise.
- Use the GitHub MCP tools (`mcp__github__create_pull_request`, `mcp__github__merge_pull_request`) — `gh` is not available.
- Branch name: keep the existing working branch (`claude/...`). Do not push directly to `main`.
- PR title: short and descriptive (under 70 chars). PR body: 1–3 bullet summary + a brief test plan.
- Merge method: squash unless the user says otherwise.
- After merging, report the PR URL and merge status to the user.

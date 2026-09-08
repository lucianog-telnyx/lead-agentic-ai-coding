# Guardrails: Forbidden Actions (STRICT)

- ❌ **NO AGENT COMMITS OR PUSHES**: Never run `git commit` (in any form, including `--amend`) or `git push` (in any form, including `--force`/`--force-with-lease`). Committing and pushing are human-only actions in every repo this hub serves — stage changes, show diffs, and stop for the human to review and commit/push themselves, even if explicitly asked to commit/push in the moment. Prefer enforcing this technically too (a `.claude/settings.json` `permissions.deny` rule blocking `Bash(git commit *)`/`Bash(git push *)`), not just by reading this file — a documented-only rule depends on the agent choosing to follow it.
- ❌ **NO DESTRUCTIVE COMMANDS**: Never execute `rm -rf /`, `git reset --hard`, or force pushes (`git push --force`) without explicit human confirmation.
- ❌ **NO ENV TAMPERING**: Never modify `.env` or configuration secrets files.
- ❌ **NO UNAPPROVED DEPENDENCIES**: Do not add major libraries or frameworks without prior approval.

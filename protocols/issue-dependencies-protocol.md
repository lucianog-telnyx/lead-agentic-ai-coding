# Protocol: Issue Dependency Graph (Blocked-by / Blocking)

## Applies When
- Creating two or more issues where one cannot reasonably start before another finishes.
- Moving an issue into, out of, or between milestones.
- Re-balancing or re-ordering a milestone's issue list for any reason (splitting an
  issue, merging two, changing execution order).

## Rule (STRICT)
- Every "must land before" / "must land after" relationship between two open issues is
  encoded as a native GitHub **issue dependency** (`blocked_by`/`blocking`) — not only
  as prose in the issue body.
- A "Depends on: #X" / "Blocks: #Y" line in an issue's References section is a summary
  for a human reading that one issue in isolation, never a substitute for the graph
  edge. Write both. The graph is authoritative — it's what the Relationships sidebar,
  a project board, a roadmap view, or another agent's `gh issue view --json
  blockedBy,blocking` will actually read; prose in a body is not queryable.
- When an issue moves between milestones, or a milestone is re-balanced, its
  dependency edges are re-checked in the same pass. Don't leave a stale edge pointing
  at something that no longer blocks it, and don't leave a real new ordering
  constraint undeclared.

## Mechanics
Plain REST — no GraphQL, no preview header:

```
GET    /repos/{owner}/{repo}/issues/{issue_number}/dependencies/blocked_by
POST   /repos/{owner}/{repo}/issues/{issue_number}/dependencies/blocked_by   { "issue_id": <int> }
DELETE /repos/{owner}/{repo}/issues/{issue_number}/dependencies/blocked_by/{issue_id}
GET    /repos/{owner}/{repo}/issues/{issue_number}/dependencies/blocking
```

- **Direction**: call the endpoint on the *dependent* issue (the one that must wait),
  naming the *prerequisite* it's blocked by — `POST .../issues/<dependent>/dependencies/blocked_by {"issue_id": <prerequisite's id>}`.
  One call creates the edge both ways; the prerequisite's `blocking` list updates
  automatically. Never call it twice from both sides.
- **`issue_id` is the issue's internal database id, not its issue number.** Resolve it
  first — `gh api repos/:owner/:repo/issues/<number> -q .id` — before posting.
  Passing the issue *number* either 404s or silently attaches the wrong node.
- **Use a typed field, not a string.** `gh api ... -f issue_id=123` sends a JSON
  string and the API rejects it: `422 Invalid property /issue_id: is not of type
  integer`. Use `-F issue_id=123` to send it as a number.
- **Check for native `gh` support before falling back to raw `gh api`.** GitHub's docs
  describe `gh issue create --blocked-by/--blocking` and
  `gh issue edit --add-blocked-by/--remove-blocked-by`, but the installed `gh` version
  may predate them. Run `gh issue edit --help` (or `gh --version`) and look for the
  flag — don't assume it's there. If it's missing, use the REST calls above.

## Verification (required after writing edges)
Read both directions back for every issue touched — don't trust the POST response
alone:

```bash
gh api repos/:owner/:repo/issues/<n>/dependencies/blocked_by -q '.[].number'
gh api repos/:owner/:repo/issues/<n>/dependencies/blocking    -q '.[].number'
```

The prerequisite's `blocking` list must contain the dependent, and the dependent's
`blocked_by` list must contain the prerequisite. A one-sided result means the wrong
`issue_id` was sent — fix it before moving on, don't leave a half-written edge.

## Removing a stale edge (rebalancing)
```
DELETE /repos/{owner}/{repo}/issues/{issue_number}/dependencies/blocked_by/{issue_id}
```
Same not-the-issue-number caveat on `issue_id`. Do this rather than leaving an
outdated edge once an issue's real prerequisites change.

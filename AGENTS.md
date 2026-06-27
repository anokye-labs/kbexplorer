# Agents — kbexplorer (docs)

This is the **documentation + showcase** repo for the kbexplorer system. It holds
**no application code** — the code lives in `kbexplorer-core`, `kbexplorer-cli`,
and `kbexplorer-template`. Keep it that way: prose, diagrams, and a showcase build
only.

## What goes here

- `docs/` — architecture documentation (the four layers, the source/affordance
  model, the provider/representation extension points).
- A deployed **showcase** of kbexplorer.

When you document behavior, link to the canonical contract in
[`kbexplorer-core`](https://github.com/anokye-labs/kbexplorer-core) rather than
re-describing types — the contract is the source of truth.

## Branch Protection

The default branch is protected:

- **Pull request required** — no direct pushes to `main`.
- **Required status checks** (strict / up-to-date) — `pr-title`,
  `check-linked-issue`, `dependency-review`.
- **Conversation resolution required** before merge.
- **0 approvals required** — agent PRs auto-merge once green via the `auto-merge`
  workflow. Force pushes and branch deletion are blocked.

Never commit directly to `main`. Never force push.

## Issue-First Workflow

**Every pull request must trace back to a GitHub Issue.**

1. Create an Issue (with a native Issue **Type**: Epic / Feature / Task / Bug).
2. Create a branch and implement.
3. Open a PR that references the issue (e.g. `Closes #12`).
4. CI goes green → the PR auto-merges (0 approvals), which closes the issue.

Use the **GraphQL API** for issue types, sub-issues, and blocked-by relationships
(the REST API does not support them). Include `GraphQL-Features: sub_issues` for
sub-issue operations.

## Verification

Check that links resolve and any showcase build succeeds before handing back
control. If you cannot fully verify a change, say so explicitly — never claim
"done" with a silent gap.

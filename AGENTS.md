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

**Check, don't assume.** A live audit (#105) found this repo has **no branch
protection ruleset on `main` at all**, despite this section previously
claiming required status checks and conversation resolution were enforced —
that claim was false. Do not rely on any protection invariant (required
checks, conversation resolution, force-push/deletion blocks) without
verifying it directly against the live repo first — e.g.
`gh api repos/anokye-labs/kbexplorer/rules/branches/main` (or the equivalent
REST/GraphQL/MCP call) — rather than trusting what this document says the
ruleset is.

Regardless of what the live ruleset does or doesn't enforce, these rules are
non-negotiable for every contributor, human or agent:

- **Never commit directly to `main`.**
- **Never force push** — to `main` or to any shared branch.

If you find the live ruleset doesn't match what's documented (or is missing
entirely), don't quietly work around the gap: it's a finding, not a
convenience. File or update an issue (see #105 for the tracked example).

## Issue-First Workflow

**Every pull request must trace back to a GitHub Issue.**

1. Create an Issue (with a native Issue **Type**: Epic / Feature / Task / Bug).
2. Create a branch and implement.
3. Open a PR that references the issue with `refs #N` — never `Closes #N` /
   `Fixes #N`. Closure is a deliberate, separate step (see
   [GitHub & Work-Item Conventions](#github--work-item-conventions)), not an
   automatic side effect of a merge.
4. CI goes green → the PR auto-merges (0 approvals). Once merged, re-verify
   the referenced issue's acceptance criteria against the now-current repo
   state, then close it citing that evidence.

Use the **GraphQL API** for issue types, sub-issues, and blocked-by relationships
(the REST API does not support them). Include `GraphQL-Features: sub_issues` for
sub-issue operations.

## GitHub & Work-Item Conventions

These conventions are tool-agnostic and apply no matter which agent or human
is working the repo:

- **Tool-agnostic GitHub interaction.** Use whichever interface is available
  and fits the task — the `gh` CLI, the REST API, the GraphQL API, or an MCP
  GitHub server. None is mandated. The one hard requirement is **GraphQL
  capability** for anything touching issue sub-issues or `blocked-by`
  dependency edges — the REST API does not expose either.
- **`refs #N`, never `closes #N`.** See Issue-First Workflow above — closure
  is deliberate, not automatic.
- **Verification before close.** Before closing any issue, re-verify its
  acceptance criteria against the **current** state of the repo — re-read the
  file, re-run the check, re-fetch the live resource — not the state at the
  time the PR was opened. Then close it citing the specific evidence (a file
  and line, a check-run URL, a fetched page) that satisfied each criterion.
- **Conventional Commits.** `type(scope): description`, per the commit
  conventions at the top of this document.
- **Workback scheduling / work breakdown.** When a program is large enough to
  need a real Epic → Feature → Task breakdown — native GitHub sub-issues plus
  `blocked-by` dependency edges sequenced into waves — don't hand-roll it.
  Use
  [`kbexplorer-template`'s `wbs-builder` skill](https://github.com/anokye-labs/kbexplorer-template/blob/main/.agents/skills/wbs-builder/SKILL.md),
  which materializes native issue types, parent/child sub-issues, and
  dependency edges via the GraphQL API and sequences work in dependency-order
  waves.

## Verification

Check that links resolve and any showcase build succeeds before handing back
control. If you cannot fully verify a change, say so explicitly — never claim
"done" with a silent gap.

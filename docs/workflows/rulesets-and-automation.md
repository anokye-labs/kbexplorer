# Rulesets & automation

How GitHub rulesets and Actions workflows act as the **management plane** for
a kbx dataset. The crux: **the host enforces; kbx declares**. kbx defines what
a healthy dataset looks like (clean generate, matching corpus, valid shapes);
the host's rulesets and required checks make those definitions binding on
every change.

![Rulesets & automation — GitHub as the management plane for a kbx dataset](rulesets-and-automation.svg)

## The flow

1. **Ruleset on `main`** — a branch ruleset blocks direct pushes and force
   pushes and requires status checks (and, where configured, conversation
   resolution) before merge. This is what makes "the delta is a proposal"
   enforceable rather than aspirational.
2. **Trace to an issue** — every PR references a typed issue (Epic / Feature /
   Task / Bug) with `refs #N`. Traceability is a convention the automation
   can check, not a habit.
3. **Actions gate the PR** — the CI workflow builds the dataset and runs the
   gates: graph integrity, drift (does the committed manifest match a clean
   `generate`?), and index freshness.
4. **kbx checks run as CI** — the checks are ordinary kbx commands run in a
   workflow, so a maintainer can reproduce any red build locally with the
   same command the bot ran.
5. **Auto-merge on green** — routine changes flow through 0-approval
   auto-merge lanes once checks pass; human review gates are configured where
   the change class warrants them (see
   [approving a change](approving-a-change.md)).
6. **Audit — don't assume** — protection is verified against the live repo
   (`gh api repos/<owner>/<repo>/rules/branches/main`), never trusted from
   documentation. A live audit
   ([#105](https://github.com/anokye-labs/kbexplorer/issues/105)) found this
   repo's own docs once claimed protections that did not exist — a gap is a
   finding to file, not a convenience to use.

## Gates & guarantees

- **No direct commits to `main`, no force pushes** — for every contributor,
  human or agent, regardless of what the live ruleset happens to enforce.
- **Drift is a red build.** A dataset whose committed artifacts disagree with
  a clean regenerate cannot merge.
- **Closure is deliberate.** Merging never auto-closes the issue; the issue
  closes only after its acceptance criteria are re-verified against the
  now-current repo state.

## Traceability

- Stories: [C3 — CI gates drift](../user-stories.md#c--keep-healthy--current--26),
  [G3 — doctor / preflight](../user-stories.md#g--operate--30),
  [G4 — build + commit index](../user-stories.md#g--operate--30).
- Conventions: the repo's `AGENTS.md` (issue-first workflow, branch
  protection, GraphQL for issue types).
- Delivered by: [kbexplorer-cli#140](https://github.com/anokye-labs/kbexplorer-cli/issues/140)
  (CI drift gate), [kbexplorer-cli#151](https://github.com/anokye-labs/kbexplorer-cli/issues/151)
  (index freshness), [kbexplorer-cli#100](https://github.com/anokye-labs/kbexplorer-cli/issues/100) /
  [#152](https://github.com/anokye-labs/kbexplorer-cli/issues/152) (doctor/preflight);
  audit tracked in [#105](https://github.com/anokye-labs/kbexplorer/issues/105).

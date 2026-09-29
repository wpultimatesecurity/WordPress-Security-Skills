# Full audit workflow

Load this only in **full review mode**: an explicit audit, pre-release review, or a
requested findings report for a whole plugin or theme. For a focused question or a
single handler, use guidance mode and skip this file.

The workflow is agent-neutral. Where the platform supports delegated agents (sub-agents,
tasks), the parent coordinates and owns the shared report; delegated agents return
results. Without delegation, run the same phases yourself in order, and re-read each
candidate from scratch before validating it.

## Execution safety

Source review is read-only. Runtime checks are for confirmation, not for expanding impact.

- Run code only on a **disposable local WordPress** (wp-env, Docker, Playground, or a
  throwaway VM) with dummy users, dummy content, and dummy secrets.
- Never send requests to a live, staging, or client site, a shared host, or a third-party
  API, and never use real accounts or production data.
- Stop at the minimum observable result: a wrong `current_user_can()` outcome, a dummy
  record modified by the wrong role, a script tag rendered unescaped, a file written to
  the wrong directory. Do not build persistence, post-exploitation, or evasion material.
- Do not install new dependencies from the network for a check; use what is present.
- If the deciding fact cannot be established from source or the local stack, record
  `needs_validation` with that fact instead of testing further.

## Phase 1 — Reconnaissance and coverage plan

1. Record the review context from the [report template](report-template.md).
2. Map the plugin: bootstrap file, modules, entry points (AJAX, REST, `admin_post_`,
   forms, shortcodes, blocks, cron, WP-CLI, `init`/`template_redirect` request handlers),
   stored data (options, meta, custom tables, files), outbound calls, and roles involved.
3. Build a **coverage ledger**: one unit per entry-point group × trust boundary × attack
   class. Use the [audit checklist](audit-checklist.md) classes plus the relevant
   [attack classes](attack-classes.md). Example unit IDs:
   `rest:/myplugin/v1/orders × subscriber→other-customer × access-control`,
   `ajax:pb_import × author→filesystem × file-upload`.
4. Give each unit one state and keep it current:

| State | Meaning |
| --- | --- |
| `planned` | Seeded, not yet reviewed. |
| `covered` | Reviewed, with the paths read and invariants checked; no candidate. |
| `candidate` | Reviewed and produced at least one candidate awaiting validation. |
| `blocked` | Cannot be settled from source or the local stack; names the missing fact. |
| `deferred` | Not reviewed in this run (budget or time); stays open work. |
| `out_of_scope` | Excluded by the agreed scope; never counted as covered. |

## Phase 2 — Hunting

Assign units in priority order: unauthenticated and low-role entry points first, then
state-changing and file/SQL/code-execution sinks, then logic and abuse classes. For
each unit, the reviewer (or delegated hunter) returns:

- the unit ID, final state, paths read, and each invariant checked with its result;
- candidates, each with location, trace, the boundary-and-result statement, and a
  proposed verdict (`confirmed` or `needs_validation`);
- hardening notes, kept separate;
- surfaces discovered that are not in the ledger, to add as new units.

Give hunters one unit or a small related group each, not the whole plugin. Do not show
hunters prior findings as examples; it anchors them on known bugs.

After each wave, run a **coverage critic**: a fresh pass over the ledger and code map
that asks which entry points, roles, or classes have no unit or only shallow evidence.
Add the gaps as units and run another wave until a critic pass finds nothing material or
the budget is spent.

## Phase 3 — Independent validation

Give every candidate to a fresh reviewer (a new agent, or yourself after re-reading the
code from the entry point without the hunter's notes) whose job is to **disprove** it:

1. Re-trace the path from the entry point to the effect; check every line cited.
2. Look for the strongest governing control: capability and meta-capability checks,
   `permission_callback`, nonce scope, `wp_kses`/escaping, `prepare()`, filters added
   elsewhere in the plugin, and WordPress core defaults.
3. Confirm the principal really lacks the authority (see the severity anchors on
   same-principal effects).
4. For a proposed `confirmed`, reproduce the minimum result on the local stack when a
   runtime fact matters; otherwise state that the proof is static.
5. Return `confirmed`, `needs_validation` (exact blocker and validation plan), or
   `rejected` (the control that stops it). Rate severity only for `confirmed`, using the
   [severity anchors](severity-anchors.md).

Merge duplicates by root cause (same function and missing control), not by symptom.

## Phase 4 — Final record check

Before writing the report, re-verify each final record once more with fresh eyes: file
and line, the entry point and input shape, conditions, the affected resource, severity
not above the demonstrated impact, and a remediation that enforces the invariant at the
last trusted decision point instead of moving it. Correct or downgrade anything that
does not hold.

## Phase 5 — Report

Fill the [report template](report-template.md) from the final records only, including
the coverage summary. Prose and verdicts must agree. The run ends in one of two states:
the report is complete, or it states plainly that the run is incomplete, why, and which
units remain `planned`, `blocked`, or `deferred`.

## Profiles and budget

| Profile | Use for | Shape |
| --- | --- | --- |
| `quick` | Small plugins, re-runs, first look | Coarse units (entry-point group × class), one hunting wave, one critic pass, one validator per candidate. Report is partial coverage. |
| `standard` | Default | The workflow as written. |
| `deep` | High-stakes or large targets | Units per subsystem and role, critic waves until clean, separate validation and final-check reviewers. |

A **scoped** run (named files, one module, or a diff between two revisions) seeds units
only for the scope and marks everything else `out_of_scope`. Scoped and `quick` runs
must describe themselves as partial coverage.

When the user sets a budget, count agent invocations. Reserve the critic pass and
validation (about one reviewer per expected candidate) **before** assigning hunters.
If the budget runs out, stop hunting, validate what exists, mark remaining units
`deferred`, and report the run as incomplete. Profiles change breadth, never the
evidence bar.

## Re-audits and prior runs

- Read prior reports first. A prior `confirmed` finding stays confirmed only if its code
  is unchanged and it is re-checked in this run; changed code makes it new work.
- A prior `rejected` item suppresses only the same unchanged claim, not its unit.
- Prior `needs_validation`, `blocked`, `deferred`, and `out_of_scope` items are current
  work, never evidence that the area is safe.
- A prior `quick` or scoped run contributes only what it recorded, never an implied
  "the rest is fine". If no prior run exists, say so in the coverage summary.

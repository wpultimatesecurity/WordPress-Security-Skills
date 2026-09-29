# WordPress security review report template

Use this contract for a bounded review, not a certification. Replace instructions
with observed evidence; keep unavailable values explicitly `Unknown`, never infer
them. Redact secrets, credentials, tokens, and personal data from excerpts, requests,
responses, and tool output before including them. Preserve enough non-sensitive
context to explain the data flow and reproduce safely in an authorized environment.

## Review context

Record this context before inventory or scanning. Identify the exact reviewed
artifact with an immutable commit, a tag resolved to a commit, or a checksum;
a branch name or an unresolved mutable tag is not an immutable revision.

| Field | Value |
| --- | --- |
| Target | Unknown |
| Revision (commit/tag/checksum) | Unknown |
| Reviewer | Unknown |
| Review date | Unknown |
| In scope | Unknown |
| Excluded | Unknown |
| Methods and tool versions | Unknown |
| Authorization for active testing | Unknown |
| Limitations | Unknown |

Include WordPress, PHP, relevant plugin/theme versions, and deployment assumptions
when known. Unknown testing authorization permits read-only review only; do not
perform active testing until authorization and boundaries are established.

## Executive summary

State the reviewed scope, material confirmed risks, and the bounded result. Include
severity counts **only for confirmed vulnerabilities**, rated from demonstrated
impact and reachability with the [severity anchors](severity-anchors.md). An empty confirmed set means no vulnerability was confirmed
within the recorded scope, not that the application is secure. Do not supply an
overall “secure” score, estimated remediation hours, or response-time promises.
Do not include CVSS/CWE values unless an actual vector/mapping and its source are
recorded; otherwise omit them rather than guessing.

## Confirmed findings

Repeat this block only after tracing the reachable operation and its governing
controls. Explain severity from evidence, including remaining exploit prerequisites.

### [SEVERITY] Category — title

- **Location:** Exact file and line range at the reviewed revision.
- **Boundary:** Lower-trust principal → input or action it controls → control that
  should stop it → affected user or resource → concrete result. If any element is
  missing, the candidate belongs under Needs validation, not here.
- **Evidence/data flow:** Entry point, attacker-controlled source, transformations,
  governing controls (or their absence), and sensitive sink. Include redacted code
  or a safe reproduction; distinguish static proof from runtime observations.
- **Exploit prerequisites:** Authentication level, capabilities, nonce acquisition,
  object ownership/restrictions, configuration, and other necessary conditions.
  Unknown prerequisites stay `Unknown`; move an incomplete exploitability trace
  to Needs validation instead of presenting it as a confirmed vulnerability.
- **Impact:** Who can do what to which resource; justify severity without expanding
  beyond the demonstrated reachability and deployment assumptions.
- **Remediation:** Concrete corrected code or configuration, with the appropriate
  WordPress APIs and fail-closed checks before privileged effects.
- **Verification:** Checks actually performed, observed results, and unexecuted
  checks explicitly labeled as proposed. Cover unauthorized, authorized, malformed,
  and alternate paths where in scope; never claim a fix was tested when only read.
- **References:** Relevant official API/security documentation and any recorded
  severity vector or classification source used.

## Needs validation

List source-grounded hypotheses blocked by a specific missing fact, including scanner
hits and incomplete traces. These are not confirmed findings: assign **no severity**
and keep them out of confirmed totals. For each item record:

- **Location and trace:** File and line range, and the source path established so far.
- **Claimed root cause:** The boundary it would cross and the possible concrete result.
- **Blocker:** The exact missing fact (for example, whether the web server executes
  `.phtml` in `uploads/`, whether a filter in another plugin runs first, or whether the
  site changes the default role).
- **Validation plan:** A safe next step: a bounded check on a disposable local
  WordPress with dummy users, and/or a configuration fact for the site owner to confirm.
  Never propose probing a live site.

Write `None` only if no items remain; do not equate a scanner match with a vulnerability.

## Rejected candidates

List candidates that a governing control disproves, so later reviews do not re-report
them: location, the claim, and the control that stops it (for example,
`current_user_can( 'manage_options' )` in the route's `permission_callback`). A missing
fact is not a rejection; that item belongs under Needs validation. Write `None` if empty.

## Hardening recommendations

Keep defense-in-depth advice without a demonstrated vulnerability separate. State
the applicable context and expected benefit; do not inflate confirmed counts with
optional controls or style changes. Write `None` if none are proposed.

## Coverage

For full reviews, summarize the coverage ledger from the
[full audit workflow](full-audit-workflow.md): unit counts by state (`covered`,
`candidate`, `blocked`, `deferred`, `out_of_scope`), the profile (`quick`, `standard`,
`deep`) and any scope restriction, the most important uncovered surfaces, and the result
of the last coverage critic pass. State whether prior reviews were used. A scoped or
`quick` run is partial coverage; never imply that unreviewed units are safe.

## Verification performed

Record methods, commands/tool versions, the environment and immutable revision,
request roles and cases exercised, and actual results. Separate source review,
scanner inventory, runtime reproduction, and post-fix verification. Identify any
failed, unavailable, or unexecuted checks and redact sensitive output. A passing
nonce test does not establish authorization; a clean scan does not prove safety.

## Residual limitations

State excluded files/entry points, unknown deployment or version assumptions,
unverified fixes, and remaining gaps. Incomplete scope cannot produce a security
certification or release sign-off. Refuse blanket “ready to ship” or “secure” claims
when only a handler or other subset was reviewed; state exactly what was established
and what further review or verification is required.

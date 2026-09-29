---
name: security-auditing-code-review
description: >
  Use when auditing or code-reviewing an existing WordPress plugin or theme for security
  issues, triaging a vulnerability report, or hardening inherited code. Provides a
  systematic methodology — locate trust boundaries, inventory sensitive sinks, trace
  their controls and data flows, then triage confirmed issues and report with fixes.
  Apply proactively before shipping or when reviewing third-party code.
compatibility: "Examples generally use PHP 7.4 syntax; check each API against target WordPress/PHP versions. Use maintained WordPress and supported PHP in production. Shell examples require their named tools."
license: MIT
metadata:
  tags: "wordpress, security, audit, code-review, vulnerability, hardening"
---

# Security auditing & code review

## When to use this skill

Use this skill when the task is to **evaluate** code rather than write a feature:

- Reviewing a PR/plugin/theme for security defects.
- Auditing inherited or third-party code before deploying it.
- Triaging a reported vulnerability or suspicious behavior.
- Producing a security findings report with severities and fixes.

This skill is the audit counterpart to the secure-coding skills. When you find an issue,
fix it using the relevant skill (`nonces-csrf-protection`, `output-escaping`,
`sql-injection-prevention`, etc.).

## Operating modes

- **Guidance mode** — a focused question, one handler, a triage of a single report, or
  a methodology question. Answer inline with the relevant parts of this skill; do not
  produce the full report or claim coverage beyond what was examined.
- **Full review mode** — an explicit audit, pre-release security review, third-party
  code assessment, or a requested findings report. Run every step below, follow the
  [full audit workflow](references/full-audit-workflow.md) (coverage ledger, hunting,
  independent validation), and use the full [report template](references/report-template.md).

In either mode, run code only on a disposable local WordPress with dummy users and
secrets; never probe a live or client site. A fact that source and the local stack
cannot settle becomes `needs_validation`.

If the request could mean either, ask one focused question before starting a full review.

## Core principles (and why they matter)

1. **Follow the data, not the file order.** Trace untrusted input from its entry point
   (`$_GET`/`$_POST`/`$_FILES`/REST) to where it is used (DB, output, filesystem). Bugs
   live on those paths.
2. **Map trust boundaries first.** Enumerate AJAX actions, REST routes, forms,
   shortcodes, cron jobs, and CLI commands. Decide which need authentication,
   authorization, CSRF protection, validation, and output escaping for their context.
3. **Trace controls, not proximity.** A nearby nonce, capability check, escaper, or
   `prepare()` call does not establish protection. Verify it governs the reachable
   operation; absence from the same line/file does not establish a vulnerability.
4. **Require a boundary and a result.** A confirmed finding names the lower-trust
   principal (anonymous visitor, subscriber, contributor, author, editor, shop manager),
   the input or action it controls, the control that should stop it, the boundary
   crossed, the affected user or resource, and the concrete result. If any part is
   missing, it is not confirmed.
5. **Separate certainty from priority.** Only confirmed findings receive severity, rated
   from demonstrated impact and reachability with the
   [severity anchors](references/severity-anchors.md). A blocked hypothesis is
   `needs_validation`, not a low-severity finding.
6. **Report with a concrete fix.** Each finding = location, what's wrong, why it matters,
   and the corrected code. A finding without a fix is half-done.
7. **Don't trust comments or names.** Verify what the code does, not what it claims.

## Step-by-step implementation

1. **Record review context before inventory/scanning:** target, immutable revision
   (commit/tag resolved to commit/checksum), reviewer, date, scope, exclusions,
   methods/tool versions, active-testing authorization, and limitations. Keep
   unavailable values explicitly Unknown; never infer them. Use the full
   [report template](references/report-template.md) throughout the review.
2. **Inventory entry points:** grep for `wp_ajax_`, `register_rest_route`, `admin_post_`,
   `add_shortcode`, `$_GET`/`$_POST`/`$_REQUEST`/`$_FILES`, form handlers. In full
   review mode, record them as coverage units (entry point × boundary × attack class)
   so the report states what was covered, blocked, deferred, or out of scope.
3. **Check applicable controls on each path:** transport-appropriate authentication
   and CSRF protection, resource-level authorization, shape/type validation,
   sanitization, output escaping, and safe query construction.
4. **Inventory sinks**, including safely guarded ones: database calls, output,
   filesystem operations, uploads, code execution, deserialization, redirects,
   and outbound requests. Follow the input and controls before labeling any hit.
   Then hunt the classes a sink sweep misses: business logic, feature abuse,
   second-order paths, and the "obvious things" in [attack classes](references/attack-classes.md).
5. **Classify before rating:** give every candidate one verdict.
   - `confirmed` — complete reachable trace, boundary, and result; severity from the
     [severity anchors](references/severity-anchors.md).
   - `needs_validation` — a source-grounded hypothesis blocked by one exact missing
     fact (host/WAF config, a filter added elsewhere, a role customization). Record the
     blocker and a safe validation plan; assign **no** severity. Scanner hits and
     incomplete traces go here.
   - `rejected` — disproved by a control that governs the path; record the control so
     the candidate is not re-reported.

   Keep defense-in-depth advice under Hardening recommendations, outside all verdicts.
6. **Report:** evidence/data flow, exploit prerequisites, impact, concrete remediation,
   verification, and references. Redact secrets/PII. Do not invent CVSS/CWE values,
   remediation hours, response promises, or an overall secure score.
7. **Re-verify** fixes across in-scope paths; distinguish executed checks from proposed
   checks. Record residual limitations and refuse blanket release sign-off or security
   certification for incomplete scope.

After recording context, use the read-only [sink inventory helper](scripts/scan-security-sinks.sh)
if Bash and ripgrep (`rg`) are available. From this skill directory:

```bash
bash scripts/scan-security-sinks.sh /path/to/plugin-or-theme
```

This is a **heuristic text search, not a security audit or vulnerability detector**.
It includes safe calls, comments, and strings; it does not prove missing controls.
No hits is not proof of safety. It searches PHP/PHTML/INC files using ripgrep
ignore rules, skipping hidden/binary files and symlinks. Dynamic/multiline calls
and other file types require separate review. Source lines can contain secrets;
keep the output local and redact before sharing. `--help` explains scope and
exit codes (0: completed, with or without hits; 2: usage/search error).

See [`references/grep-patterns.md`](references/grep-patterns.md) for additional searches
and [`references/audit-checklist.md`](references/audit-checklist.md) for the full review pass.

### Supporting references

| Reference | Load when |
| --- | --- |
| [WordPress security audit checklist](references/audit-checklist.md) | Performing the full manual audit pass across entry points, controls, and sinks. |
| [Audit grep patterns](references/grep-patterns.md) | Expanding the entry-point and sink inventory with additional heuristic searches. |
| [WordPress security review report template](references/report-template.md) | Recording review context before inventory and reporting evidence, classification, verification, and limitations. |
| [WordPress attack classes](references/attack-classes.md) | Hunting access-control, business-logic, feature-abuse, second-order, wildcard, and obvious-exposure classes beyond the sink sweep. |
| [Full audit workflow](references/full-audit-workflow.md) | Running full review mode: execution safety, coverage ledger, hunting waves, independent validation, profiles, budget, and re-audits. |
| [Severity anchors](references/severity-anchors.md) | Assigning severity to a confirmed finding or checking that a rating matches demonstrated impact. |

## Common AI mistakes / anti-patterns

### Mistake 1 — Reviewing for style, missing the security sink

```php
// Reviewer comment: "rename $q to $query for clarity" ← misses the actual bug:
$rows = $wpdb->get_results( "SELECT * FROM t WHERE id = " . $_GET['id'] ); // SQL injection
```

Flag the **injection** first. Cosmetic notes never outrank a Critical finding.

### Mistake 2 — Assuming a nonce implies authorization (or vice versa)

```php
// A nonce check is present, so the reviewer marks it "secure" —
check_admin_referer( 'act' );
delete_user( absint( $_POST['id'] ) ); // ❌ still missing current_user_can()
```

Verify **both** controls independently on every state-changing path.

### Mistake 3 — Trusting `sanitize_*` as if it were escaping (or the reverse)

```php
// Input was sanitized on save, so output is assumed safe — but context differs:
echo '<a href="' . get_option( 'my_url' ) . '">'; // ❌ needs esc_url on output
```

Sanitize-on-input and escape-on-output are separate; check both ends.

### Mistake 4 — Marking everything Critical (or burying real issues)

Inflated severity destroys signal. Rate by impact × reachability: an unauthenticated RCE
is Critical; a self-XSS reachable only by an admin editing their own profile is Low/Info.
Apply the [severity anchors](references/severity-anchors.md): overall severity never
exceeds the demonstrated impact.

### Mistake 5 — Reporting the problem without the fix

```text
❌ "Line 42 is vulnerable to XSS."
✅ "Line 42: $name echoed unescaped into HTML (stored XSS, High).
    Fix: echo esc_html( $name );"
```

### Mistake 6 — Reporting a principal's own authority as a vulnerability

```php
// Reviewer: "Stored XSS — post content is saved without wp_kses_post()."
// But only users with unfiltered_html (administrators/editors on single site) reach it:
if ( current_user_can( 'unfiltered_html' ) ) {
	update_post_meta( $post_id, '_custom_html', wp_unslash( $_POST['custom_html'] ) );
}
```

Raw HTML from a principal WordPress already trusts with `unfiltered_html` crosses no
boundary. Report it only if a lower-trust principal (a contributor, or a multisite site
admin without `unfiltered_html`) can reach the same write, or if the result affects
someone the writer could not already affect.

### Mistake 7 — Guessing the deployment

```text
❌ "Exploitable: the site has no WAF."   ❌ "Not exploitable: hosts block PHP in uploads."
✅ needs_validation — blocker: whether uploads/ executes .phtml on the target server.
   Validation plan: owner confirms the web-server handler for uploads/, or reproduce
   on a disposable local stack with the recorded server config.
```

Host rules, WAF/CDN behavior, `wp-config.php` constants such as `DISALLOW_FILE_EDIT`,
and role customizations are real controls. If the decision depends on one that is not
in the reviewed source, do not assume it present or absent; record `needs_validation`
with the exact missing fact.

### Mistake 8 — Checklist deviations presented as findings

```text
❌ "[Medium] Plugin directory lacks index.php silence file."
❌ "[Low] needs_validation: possible SSRF if the host allows internal requests."
✅ Hardening: add index.php (no reachable disclosure demonstrated).
✅ needs_validation (no severity): an editor-set webhook URL reaches wp_remote_get();
   blocker: whether internal hosts are reachable from the target server.
```

A missing best practice with no affected principal or resource is a hardening note,
not a finding. A `needs_validation` item never carries severity.

## Correct code examples

Use the full [report template](references/report-template.md) for the review.
Keep this compact skeleton for each **confirmed** finding:

```text
[SEVERITY] Category — title
Location: Exact file:line at the reviewed revision.
Evidence/data flow: Reachable source, governing controls, and sensitive sink.
Exploit prerequisites: Required identity, nonce access, object restrictions, configuration.
Impact: Demonstrated consequences and evidence-based severity rationale.
Remediation: Concrete corrected code/configuration with fail-closed controls.
Verification: Executed checks and results; proposed checks explicitly unexecuted.
References: Relevant official API/security sources.
```

## Checklist

- [ ] Review context records scope, immutable revision, reviewer/date, methods/versions,
  testing authorization, exclusions, and limitations; unavailable values remain Unknown.
- [ ] All in-scope entry points inventoried; no claim extends beyond reviewed paths.
- [ ] Full reviews report coverage (covered, blocked, deferred, out of scope); partial
  or `quick` runs say so.
- [ ] Runtime checks ran only on a disposable local WordPress with dummy data.
- [ ] State-changing paths checked independently for applicable CSRF and authorization controls.
- [ ] Each output checked for context-correct escaping.
- [ ] Each custom query checked for `$wpdb->prepare()`.
- [ ] File/path operations checked for allowlisting and traversal.
- [ ] Dangerous sink hits traced before classifying them as vulnerabilities.
- [ ] Each confirmed finding names the lower-trust principal, input, bypassed control,
  crossed boundary, affected resource, and concrete result.
- [ ] Every candidate has one verdict: `confirmed`, `needs_validation` (exact blocker and
  validation plan), or `rejected` (the governing control recorded).
- [ ] Only confirmed findings carry severity, rated with the severity anchors and never
  above demonstrated impact; `needs_validation` items and hardening stay outside totals.
- [ ] Each finding records location, evidence/data flow, exploit prerequisites, impact,
  concrete remediation, verification, and references.
- [ ] Executed checks and results distinguished from proposed checks and unverified fixes.
- [ ] Secrets/PII redacted; residual scope limitations explicit; no blanket sign-off or certification.

## Official references

- [Plugin Security — Plugin Handbook](https://developer.wordpress.org/plugins/security/)
- [Data Validation](https://developer.wordpress.org/apis/security/data-validation/)
- [Escaping Data](https://developer.wordpress.org/apis/security/escaping/)
- [WPCS — WordPress Coding Standards (security sniffs)](https://github.com/WordPress/WordPress-Coding-Standards)
- [OWASP Web Security Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)
- [OWASP Top Ten](https://owasp.org/www-project-top-ten/)

# Severity anchors for WordPress findings

Use these anchors only for `confirmed` findings: a complete reachable trace naming the
lower-trust principal, the bypassed control, the affected resource, and the concrete
result. `needs_validation` and `rejected` candidates, scanner hits, and hardening
recommendations never receive severity.

Rate the demonstrated result, not the worst outcome the bug class could produce. The
overall severity never exceeds the demonstrated impact; uncommon prerequisites lower it.

## Anchors

| Severity | Anchor | WordPress examples |
| --- | --- | --- |
| Critical | An unauthenticated actor gains code execution, full database access, or takeover of arbitrary accounts. | Unauthenticated arbitrary file upload to an executable path; `nopriv` AJAX SQL injection that reads `wp_users`; unauthenticated password reset or login for any user ID; unauthenticated option update that sets `users_can_register` and `default_role` to `administrator`. |
| High | An actor fully defeats an explicit security control with real consequences. | Subscriber-to-administrator privilege escalation; REST route whose `permission_callback` returns `true` for a destructive action; stored XSS that executes in an administrator's session from a contributor or anonymous input; authenticated RCE for a role that does not already have code-level access; reading another customer's WooCommerce orders; cross-site data access in multisite. |
| Medium | A real boundary violation with limited blast radius, uncommon preconditions, or effects confined to a narrow resource set. | CSRF on a settings change that requires a logged-in administrator to visit an attacker page; IDOR limited to non-sensitive metadata of other users' drafts; reflected XSS that needs user interaction and a guessable nonce; SSRF limited to blind GET requests with no response data. |
| Low | Disclosure of non-secret internals, or an effect requiring sustained effort for minimal gain. | Full path disclosure through a PHP notice; plugin version exposed on an unauthenticated endpoint; user enumeration where usernames are already public in author archives. |
| Informational | A confirmed but minimal-impact observation, mainly useful as a prerequisite inside a larger finding. | A nonce leaked to subscribers where every action it protects also checks a capability they lack. |

## High versus medium

Ask: does the demonstrated result **fully defeat** an explicit control for an action
with real consequences, or only **weaken** it? A full bypass is High; a narrowed or
conditional bypass is Medium. If you cannot state the concrete damage, the severity is
lower than it feels.

## Reachability adjustments

Adjust within or below the anchor from what the trace establishes:

- **Required role.** Anonymous > subscriber/customer (open registration is common, but
  record whether it is enabled when known) > contributor/author > editor >
  administrator. An effect available only to a principal who already has that authority
  (for example, an administrator with `unfiltered_html` or `edit_plugins`) crosses no
  boundary and is not a finding.
- **Nonce acquisition.** Record where the attacker obtains a valid nonce. A nonce
  printed to every logged-in user protects less than one printed only on an admin page.
- **Non-default configuration.** A result that requires a setting off by default, a
  specific add-on, or a server behavior outside the reviewed source is either rated
  lower with the condition stated, or is `needs_validation` if the condition is unknown.
- **User interaction.** Requiring a privileged user to click a crafted link lowers
  likelihood, not impact; state both.

## Rules

- Do not invent CVSS vectors, CWE mappings, or scores; include them only with the actual
  vector or mapping and its recorded source.
- Do not raise severity for defense-in-depth gaps, missing best practices, or effects on
  the acting user alone.
- Do not strengthen a PHP warning into code execution, a slow query into a site outage,
  or a same-principal action into privilege gain.
- When severity is uncertain because a fact is missing, the finding is not confirmed:
  record `needs_validation` with that fact.

# WordPress attack classes beyond the sink sweep

The [audit checklist](audit-checklist.md) covers per-entry-point controls and dangerous
sinks. Scanners and sink greps miss the classes below, because the vulnerable code often
calls safe APIs correctly and fails only in its logic. Pick the classes that fit the
target, split large plugins per subsystem, and apply the same bar to every candidate: a
lower-trust principal, a bypassed control, an affected resource, and a concrete result.
Anything short of that is `needs_validation` or hardening.

## Access control beyond "is there a check"

Verify the check is the **right** capability for the **right** object on **every** path:

- Two routes to the same state change with different checks: an AJAX handler checks
  `manage_options` while the REST route for the same action checks only `edit_posts`.
- Authentication without authorization: `is_user_logged_in()` or a nonce as the only
  gate on an action that subscribers must not perform.
- Generic capability where a meta capability is required: `current_user_can( 'edit_posts' )`
  instead of `current_user_can( 'edit_post', $post_id )`, letting an author edit another
  author's post.
- Request fields overriding intended restrictions: `role`, `user_id`, `post_author`,
  `post_status`, `blog_id`, or `meta_input` accepted from the request body.
- Bulk, batch, import, and export actions that authorize once instead of per item.
- REST `permission_callback` that checks the collection but not the item in `update_item`
  / `delete_item`, or a `__return_true` on a route that writes.

## Business logic

Standard scanners cannot find these; walk each multi-step workflow by hand.

- **Skipped or replayed steps:** approval, checkout, email confirmation, or onboarding
  steps reachable directly by URL or action name; completing a finished flow twice.
- **Races with business impact:** coupon or gift-card redemption, stock decrement,
  credit balances, or vote counts using read-then-`update_post_meta()` without locking,
  so concurrent requests double-apply (`woocommerce-security` covers order specifics).
- **Numeric manipulation:** negative quantities, zero prices, float precision, and
  string/number coercion in totals, limits, and refunds.
- **Time-based logic:** expiry checks against local time instead of UTC, transients used
  as the sole expiry for security tokens, off-by-one at boundary moments.
- **Fail-open defaults:** behavior when an option is missing, a feature is toggled off,
  a required add-on is deactivated, or a migration is half-applied.

## Feature abuse and data leakage

Legitimate features used against the site:

- **Export as exfiltration:** CSV/JSON exports, backups, or debug-info reports that a
  lower role can trigger, or that include other users' data, private/draft posts,
  password hashes, API keys, or order details.
- **Import as injection:** importers that overwrite existing records, set `post_author`
  or roles, write options, or bypass the sanitization of the normal UI.
- **Search, filter, and sort as oracles:** REST `search`, `orderby`, `meta_key`, or
  `status` parameters that confirm the existence or value of content the user cannot
  read; custom `WP_Query` args passed through from the request.
- **User enumeration:** `?author=N` redirects, `/wp/v2/users`, distinct login or
  password-reset errors, and registration responses. Rate against the anchors: usernames
  already public in author archives are Low at most.
- **Preview and draft leakage:** preview links or tokens that unlock more than one item;
  private or draft content exposed through REST listings, feeds, sitemaps, search, or
  `wp_ajax_nopriv_` responses; private pages cached by a page cache and served to
  anonymous visitors.
- **Callbacks as SSRF:** user-settable webhook, avatar, import, or oEmbed URLs fetched
  server-side (see `http-api-ssrf-prevention`).

## Second-order and chained paths

- **Stored, then trusted:** a value sanitized for one context is later used in another:
  an option rendered into a `<script>` block, a post meta value placed into SQL
  `ORDER BY`, a term slug used as a file path, a stored URL passed to `wp_redirect()`.
- **Cross-component assumptions:** component A validates for its own use and passes the
  value on; component B assumes a stronger guarantee (truncation, type, site scope).
- **Capability growth:** a role change, an application password, a delegated REST
  token, or a multisite `switch_to_blog()` that leaves a principal with more authority
  than the final action should allow.
- **Ordering windows:** capability checked at request start but the object changes before
  use; revoked users whose sessions, application passwords, or cached capabilities stay
  valid; deleted content restored from trash or revisions without re-checking ownership.
- **Chains:** connect confirmed pieces only. A leaked nonce plus a handler that checks
  only that nonce is a finding; a leaked nonce alone usually is not.

## Resource exhaustion

Unbounded queries, per-request option or transient growth, cron floods, and anonymous
paths that spend paid API quota. Use [resource exhaustion](resource-exhaustion.md) for
the classes, the finding bar, and local-only validation rules.

## Wildcard

No assigned category. Read the code that looks boring or disconnected from security:

- Half-finished, legacy, compatibility, or "debug" features, and code paths left from
  earlier versions (`*_old`, `*_legacy`, commented-out checks).
- Actions the UI never sends but the handler accepts: extra parameters, unused REST
  methods, hidden AJAX actions registered for both `wp_ajax_` and `wp_ajax_nopriv_`.
- Features combined in ways not designed together: import + multisite, preview + cache,
  REST + application passwords, shortcodes rendered in comments or widgets.
- Comments claiming something is safe: verify the claim.
- Tests: which edge cases are covered, and which are not?
- Git history: reverted security fixes, removed checks, secrets committed then deleted.

## Obvious things

Literal and thorough; verify each hit's full path before reporting it.

- Hard-coded credentials, API keys, license keys, or `-----BEGIN` blocks in source.
- `TODO`/`FIXME`/`HACK` comments mentioning auth, nonce, sanitize, or permission.
- Debug toggles reachable by query parameter, cookie, or header in production.
- Unprotected diagnostic endpoints: `?debug=1`, `phpinfo()`, log viewers, status routes.
- Committed `.env`, `*.sql` dumps, `*.log`, `*.pem`, or backup archives in the package.
- Direct-access PHP files without `defined( 'ABSPATH' ) || exit;` that execute logic.
- Open redirects: `redirect_to`, `return`, `next`, or `url` parameters passed to
  `wp_redirect()` instead of `wp_safe_redirect()` with an allowlist.
- CORS headers reflecting any origin with credentials (see `security-headers-csp`).
- Error responses returning SQL errors, stack traces, or absolute paths.
- Vendored libraries with known vulnerabilities (see `dependency-supply-chain-security`).

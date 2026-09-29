# Resource exhaustion and availability in WordPress code

Load this when a review covers public or low-role entry points that do expensive work,
store data, schedule jobs, or spend a paid API quota. The finding bar is the same as
elsewhere: a lower-trust principal, a missing bound or quota, a shared resource, and a
demonstrated result. "This query could be slow" is not a finding.

## What counts

Report a candidate only when **all** hold:

- A lower-trust principal (usually anonymous or subscriber) controls the size, count, or
  frequency of the work.
- The work consumes something **shared**: PHP workers, database time, disk, the options
  table, the cron queue, outbound connections, or operator-paid API spend.
- No source-visible bound stops it: capability check, per-user or per-IP limit, cache,
  maximum size, pagination cap, or deduplication.
- The effect on other users or the operator is concrete (other requests fail or stall,
  disk fills, a bill grows), not only the attacker's own request being slow.

Validate only on a disposable local stack with explicit, small limits. **Never** load-test
a live, staging, or shared site. If the effect depends on hosting limits (PHP workers,
`max_execution_time`, object cache present or not, CDN rate limiting), the result is
`needs_validation` with that fact named.

Rate with the [severity anchors](severity-anchors.md): an unauthenticated request that
reliably stops a shared site can be High; a slow page for one user is not a finding.

## Computational amplification

- `posts_per_page => -1`, `nopaging`, `number => ''`, or a request-controlled
  `per_page` with no cap in `WP_Query`, `get_users()`, `get_terms()`, or custom SQL.
- REST collection routes whose `per_page` schema lacks `maximum`, or custom routes that
  ignore it.
- Unindexed `meta_query` / `LIKE '%…%'` searches reachable from `wp_ajax_nopriv_` or
  public REST routes, with no cache.
- Request-controlled regular expressions, or user input inside a pattern with nested
  quantifiers (catastrophic backtracking).
- Image processing, PDF generation, or zip creation on request-controlled dimensions
  or counts.
- XML-RPC `system.multicall` or batch REST (`/batch/v1`) wrapping expensive calls.

## Accumulation

- Options written per request or per visitor, especially with autoload enabled
  (`add_option()` / `update_option()` defaulting to autoload), growing the query run on
  every page load.
- Transients keyed on user input (`'my_cache_' . $_GET['q']`) without an object cache,
  creating unbounded rows in `wp_options`.
- Anonymous submissions (forms, logs, analytics, 404 tracking) stored without size
  limits, rate limits, or retention cleanup.
- Uploads or imports without size, count, or decompressed-size limits (zip bombs).

## Scheduling and queues

- `wp_schedule_single_event()` or Action Scheduler jobs created from anonymous requests
  without deduplication (`wp_next_scheduled()`), flooding the cron queue.
- Cron callbacks that process unbounded batches instead of chunks with a time budget.
- Loopback or self-requests (`wp_remote_post( admin_url( 'admin-ajax.php' ) )`) triggered
  per visitor.

## Operator-paid spend

- Anonymous or subscriber access to endpoints that call paid APIs (AI/LLM completion,
  SMS, geocoding, email sending) without per-user quotas, caching, or capability checks.
- Outbound requests with no timeout (`'timeout'` defaults are short, but custom cURL or
  raised values can hold workers).

## Fix patterns

- Cap every request-controlled count (`min( absint( $per_page ), 100 )`), and use REST
  schema `maximum`.
- Require a capability for expensive operations; keep `nopriv` handlers cheap.
- Cache expensive public results under a bounded key set, not raw user input.
- Rate-limit by user and IP with a transient counter; deduplicate scheduled events.
- Set `autoload` to `false` (or `'off'`) for large or rarely read options.
- Enforce size limits, retention cleanup, and chunked batch processing with time budgets.

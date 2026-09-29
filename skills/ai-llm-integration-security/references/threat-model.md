# AI feature threat model

Use this to map an AI feature before building or reviewing it. For each row, record who
can write the source or reach the tool, and which control governs it.

## Injection sources

Anything that enters the prompt can carry instructions. Rate each source by the lowest
role that can write it.

| Source | Lowest writer on a typical site | Notes |
| --- | --- | --- |
| Comments, reviews, form entries, support tickets | Anonymous | Highest risk; assume hostile. |
| Post content, excerpts, custom fields | Contributor/author | Also imported content and revisions. |
| Order notes, customer profile fields | Customer | WooCommerce and membership plugins. |
| Uploaded files (PDF, DOCX, images with text) | Author or customer | Text extraction feeds the model hidden text. |
| Fetched web pages, feeds, oEmbed | Any third party | Also redirects from allowed hosts. |
| Earlier tool results and chat history | Depends on the tool | A tool that reads comments imports their instructions. |
| User profile fields (display name, bio) | Subscriber | Often placed in system prompts. |

A source written by a lower role than the user running the feature is a privilege boundary:
its instructions must not cause actions beyond what the lower role could do.

## Tool risk tiers

| Tier | Examples | Required controls |
| --- | --- | --- |
| Read, public data | Search published posts, get store hours | Capability check if not public; size limits. |
| Read, private data | Get order, list users, read drafts | Per-object capability; minimum fields; no secrets. |
| Write, reversible | Save draft, add note, trash post | Per-object capability; argument validation; audit log. |
| Destructive or outward | Delete, publish, send email, refund, change role | All of the above plus server-verified single-use user confirmation. |
| Code or configuration | Install/activate plugins, edit files, change settings, run SQL | Do not expose to model-driven tools. |

## Review checks

- Which entry points reach the model, and which roles can reach them?
- For each prompt source: which role writes it, and is it lower than the running user?
- For each tool: tier, `permission_callback`, argument validation, and whether a flow that
  reads lower-role content can call it.
- Where does output land (HTML, meta, email, options, queries), and how is it escaped or
  validated there?
- Is retrieval filtered by the current user's read permissions, and are indexes updated
  when content becomes private or is deleted?
- Where are provider keys stored, and can any response or script expose them?
- What limits cap anonymous and low-role usage and total spend?
- For MCP exposure: which WordPress user does each client authenticate as, and which
  abilities does the server expose to it?

A finding still needs a lower-trust principal, a bypassed control, an affected resource,
and a concrete result. "The model might be tricked" without a reachable tool or sink is
hardening advice, not a vulnerability.

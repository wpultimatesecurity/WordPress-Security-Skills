# WordPress Security Skills

[![Validate](https://github.com/wpultimatesecurity/WordPress-Security-Skills/actions/workflows/validate.yml/badge.svg)](https://github.com/wpultimatesecurity/WordPress-Security-Skills/actions/workflows/validate.yml)
[![Latest release](https://img.shields.io/github/v/release/wpultimatesecurity/WordPress-Security-Skills)](https://github.com/wpultimatesecurity/WordPress-Security-Skills/releases/latest)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Skills: 27](https://img.shields.io/badge/skills-27-informational.svg)](#skills)
[![Agent Skills spec](https://img.shields.io/badge/Agent%20Skills-spec-lightgrey.svg)](https://agentskills.io/specification)

Security guidance for AI coding agents that write or review WordPress plugins and themes.

AI agents often produce WordPress code that works but skips the basics: nonce checks,
capability checks, output escaping, prepared SQL queries. These skills give agents such as
Claude Code, Cursor, Codex, OpenCode, and Gemini CLI the rules, examples, and checklists they
need to write secure code by default and to audit existing code properly.

**What you get**

- 27 focused skills, one per security topic, each with wrong-vs-right code examples and a checklist.
- A structured audit workflow that separates confirmed vulnerabilities from open questions and
  hardening advice, with evidence-based severity.
- Links to the official WordPress documentation for every API the skills rely on.

**What this is not**

- Not a WordPress plugin, a scanner, or runtime protection for a website.
- Not a guarantee of secure output. Skills improve the instructions an agent follows; you still
  review the code it produces.

[Quick start](#quick-start) · [Using the skills](#using-the-skills) · [Skills](#skills) ·
[Manual installation](#manual-installation) · [Contributing](CONTRIBUTING.md) ·
[Report a vulnerability](SECURITY.md)

## Quick start

### Option 1: install with the skills CLI

Requires Node.js. Run this in the project where you want the skills:

```bash
# Choose skills and install scope interactively
npx skills add wpultimatesecurity/WordPress-Security-Skills

# Install all skills without prompts
npx --yes skills add wpultimatesecurity/WordPress-Security-Skills
```

### Option 2: ask your agent to install them

Open your AI coding agent in your WordPress project and paste:

```text
Install the WordPress security skills from
https://github.com/wpultimatesecurity/WordPress-Security-Skills for this project.

Use this agent's supported skills location (project scope unless I say global).
Copy each skill directory with its references. Keep my existing skills and ask
before replacing anything with the same name. Do not change my WordPress site,
commit, or push. When done, list the installed skills and where they are, and
confirm the agent can discover them.
```

Add "install them globally" if you want the skills in every project. Pick one scope to avoid
duplicate copies.

### Check that it worked

Restart or reload your agent if needed, then ask:

```text
Which WordPress security skills can you see, and where were they loaded from?
```

If your agent does not support skills, ask it to read the relevant `SKILL.md` file and its
linked references before starting work.

## Using the skills

Once installed, agents load the right skill automatically based on the task. You can also
name a skill directly.

**Writing new code.** Start with
[`secure-plugin-development`](skills/secure-plugin-development/SKILL.md). It sets the secure
baseline and routes the agent to the focused skills it needs.

```text
Add a REST endpoint that lets editors bulk-update post meta. Follow the
WordPress security skills.
```

**Reviewing existing code.** Start with
[`security-auditing-code-review`](skills/security-auditing-code-review/SKILL.md).

```text
Use the WordPress security skills to review this plugin. Start read-only:
map entry points and permission checks, then report confirmed findings with
file/line, who can exploit each one, impact, and a fix. List open questions
and hardening suggestions separately. Propose a plan before editing anything.
```

**Tips for better results**

1. **Give context.** Your WordPress and PHP versions, plugin or theme, multisite or WooCommerce,
   and which user roles matter. Never paste production credentials.
2. **Ask for evidence, not a score.** Each finding should state the file and line, the role that
   can exploit it, the impact, and how to verify the fix.
3. **Keep changes reversible.** Review the diff, test locally or on staging, and keep a backup
   before deploying. Installing skills does not authorize an agent to change a live site.

## Skills

Each skill targets a specific mistake AI agents make in WordPress code.

### Start here

| Skill | Use it when | What it prevents |
| --- | --- | --- |
| [`secure-plugin-development`](skills/secure-plugin-development/) | Starting any new plugin or feature | Missing `ABSPATH` guards, work done before checks, hand-rolled SQL and HTTP instead of core APIs |
| [`security-auditing-code-review`](skills/security-auditing-code-review/) | Auditing or reviewing existing code | Style-only reviews that miss real bugs, inflated severity, findings without fixes |

### Input, output, and queries

| Skill | Use it when | What it prevents |
| --- | --- | --- |
| [`input-sanitization-validation`](skills/input-sanitization-validation/) | Reading `$_GET`, `$_POST`, or any request data | Missing `wp_unslash()`, wrong sanitizer, trusting `strip_tags()`, no validation |
| [`output-escaping`](skills/output-escaping/) | Printing anything into HTML, attributes, URLs, or JavaScript | Cross-site scripting (XSS) from unescaped or wrongly escaped output |
| [`sql-injection-prevention`](skills/sql-injection-prevention/) | Writing custom `$wpdb` queries | SQL injection from concatenated input or misused `prepare()` |
| [`object-injection-deserialization`](skills/object-injection-deserialization/) | Handling serialized data | `unserialize()` on attacker-controlled data |

### Access control and authentication

| Skill | Use it when | What it prevents |
| --- | --- | --- |
| [`capability-permission-checks`](skills/capability-permission-checks/) | Deciding who may do something | Role checks instead of capabilities, hidden UI instead of real checks, missing per-object checks |
| [`nonces-csrf-protection`](skills/nonces-csrf-protection/) | Handling forms and state-changing requests | Cross-site request forgery (CSRF), and nonces used as a substitute for permission checks |
| [`authentication-session-security`](skills/authentication-session-security/) | Building login, 2FA, or session features | Custom password checks, unthrottled logins, sessions that survive password or role changes |

### Entry points

| Skill | Use it when | What it prevents |
| --- | --- | --- |
| [`rest-api-security`](skills/rest-api-security/) | Registering REST routes | `__return_true` permission callbacks on writes, unvalidated arguments |
| [`ajax-security`](skills/ajax-security/) | Adding `admin-ajax.php` handlers | Missing nonces, privileged actions exposed to logged-out users |
| [`settings-options-security`](skills/settings-options-security/) | Building settings pages or storing options | Settings saved without sanitization, options printed unescaped |
| [`shortcode-block-security`](skills/shortcode-block-security/) | Writing shortcodes or dynamic blocks | Unescaped attributes, trusting `shortcode_atts()` to sanitize |
| [`gutenberg-block-editor-security`](skills/gutenberg-block-editor-security/) | Building block editor features | Unescaped render callbacks, REST fields without permission checks, unsafe editor JavaScript |
| [`wp-cli-security`](skills/wp-cli-security/) | Writing WP-CLI commands | SQL built from CLI arguments, assumed admin context, printed secrets |
| [`cron-background-job-security`](skills/cron-background-job-security/) | Scheduling cron or background jobs | Permission checks that cannot work in cron, secrets in job arguments |

### Files and outbound requests

| Skill | Use it when | What it prevents |
| --- | --- | --- |
| [`file-upload-security`](skills/file-upload-security/) | Accepting file uploads | Executable uploads, trusted client MIME types, path traversal |
| [`filesystem-security`](skills/filesystem-security/) | Reading, writing, or deleting files | File paths built from input, including user-controlled files |
| [`http-api-ssrf-prevention`](skills/http-api-ssrf-prevention/) | Fetching remote URLs | Server-side request forgery (SSRF) to internal hosts |

### Data and secrets

| Skill | Use it when | What it prevents |
| --- | --- | --- |
| [`user-data-protection-privacy`](skills/user-data-protection-privacy/) | Storing personal data | No GDPR export/erase support, full IP storage, personal data shown to the wrong users |
| [`secrets-credentials-management`](skills/secrets-credentials-management/) | Handling API keys, tokens, or passwords | Hardcoded keys, reversible password storage, tokens in logs |

### Platforms and integrations

| Skill | Use it when | What it prevents |
| --- | --- | --- |
| [`multisite-security`](skills/multisite-security/) | Code runs on multisite networks | Site and network permissions confused, untrusted `blog_id`, data leaking between sites |
| [`woocommerce-security`](skills/woocommerce-security/) | Extending WooCommerce | Orders exposed to the wrong users, stored payment data, leaked customer data |
| [`ai-llm-integration-security`](skills/ai-llm-integration-security/) | Adding AI or LLM features | Unescaped model output, AI tools without permission checks, provider keys in the browser |

### Site hardening and supply chain

| Skill | Use it when | What it prevents |
| --- | --- | --- |
| [`wp-hardening-best-practices`](skills/wp-hardening-best-practices/) | Configuring `wp-config.php` and the server | Debug output in production, the file editor left on, executable uploads, loose permissions |
| [`security-headers-csp`](skills/security-headers-csp/) | Setting HTTP headers, CSP, CORS, or cookies | Missing headers, permissive CSP, CORS that trusts any origin |
| [`dependency-supply-chain-security`](skills/dependency-supply-chain-security/) | Adding libraries, CDN assets, CI, or updaters | Outdated libraries, unpinned scripts, remotely loaded code, leaked release secrets |

### How each skill is organized

Every skill has the same sections: when to use it, core principles, step-by-step instructions,
common AI mistakes with wrong and corrected code, a checklist, and official references. Longer
material lives in each skill's `references/` folder and is loaded only when needed. Examples
are illustrations, not a drop-in plugin: adapt prefixes, capabilities, and storage to your
project.

## Manual installation

Skills are plain folders. Copy the ones you want, or the whole `skills/` folder, into your
agent's skills directory. Run these commands from a clone of this repository.

<details>
<summary><strong>Claude Code</strong></summary>

```bash
# All your projects
mkdir -p ~/.claude/skills && cp -r skills/* ~/.claude/skills/

# This project only (can be committed with the project)
mkdir -p .claude/skills && cp -r skills/* .claude/skills/
```

Claude Code reads `~/.claude/skills/<name>/SKILL.md` and `.claude/skills/<name>/SKILL.md`.
See the [Claude Code skills docs](https://code.claude.com/docs/en/skills).

</details>

<details>
<summary><strong>Cursor</strong></summary>

```bash
mkdir -p .cursor/skills && cp -r skills/* .cursor/skills/
```

Cursor also reads `.agents/skills/`. See the [Cursor skills docs](https://cursor.com/docs/skills).

</details>

<details>
<summary><strong>OpenCode</strong></summary>

```bash
# This project only
mkdir -p .opencode/skills && cp -r skills/* .opencode/skills/

# All your projects
mkdir -p ~/.config/opencode/skills && cp -r skills/* ~/.config/opencode/skills/
```

OpenCode also reads the Claude Code locations. See the
[OpenCode skills docs](https://opencode.ai/docs/skills).

</details>

<details>
<summary><strong>Other agents (Codex, Gemini CLI, and others)</strong></summary>

Any agent that implements the [Agent Skills specification](https://agentskills.io/specification)
can load these skills. Many read `.claude/skills/` or `.agents/skills/`, so the Claude Code
commands above often work as they are. Check your agent's documentation for its skills path.

</details>

To update, pull the latest version of this repository, review the changes, and copy the skills
again. Note the installed commit if you need reproducible results.

## Compatibility

- Examples use PHP 7.4 syntax as a baseline. Newer WordPress or PHP requirements are noted
  where they apply. Use a supported PHP version and a maintained WordPress release in
  production.
- Skills follow the open [Agent Skills specification](https://agentskills.io/specification):
  each is a folder with a `SKILL.md` file and a `references/` folder.

## Limitations

- The skills guide development and review. They do not enforce security at runtime or prove
  that a site is secure.
- Automated checks in this repository validate structure, PHP syntax, and coding standards.
  They are not an independent security audit of every example.
- Always check API behavior against the WordPress and PHP versions you support.

## Contributing

Fixes and new skills are welcome. See [CONTRIBUTING.md](CONTRIBUTING.md) for the skill format,
writing rules, and the requirement to verify every API against developer.wordpress.org.

This content is written with AI assistance under human direction. See
[docs/ai-authorship.md](docs/ai-authorship.md) for what our validation does and does not
establish.

To report a vulnerability or an insecure example, follow [SECURITY.md](SECURITY.md).

## Official WordPress security references

- [Security — Common APIs Handbook](https://developer.wordpress.org/apis/security/)
- [Plugin Security — Plugin Handbook](https://developer.wordpress.org/plugins/security/)
- [Data Validation](https://developer.wordpress.org/apis/security/data-validation/) ·
  [Sanitizing](https://developer.wordpress.org/apis/security/sanitizing/) ·
  [Escaping](https://developer.wordpress.org/apis/security/escaping/) ·
  [Nonces](https://developer.wordpress.org/apis/security/nonces/)
- [Hardening WordPress](https://developer.wordpress.org/advanced-administration/security/hardening/)
- [WordPress Coding Standards](https://github.com/WordPress/WordPress-Coding-Standards)
- [OWASP Top Ten](https://owasp.org/www-project-top-ten/) ·
  [OWASP Cheat Sheet Series](https://cheatsheetseries.owasp.org/)

## Acknowledgments

The audit workflow in `security-auditing-code-review` (verdicts, severity anchors, coverage
tracking, attack classes), and the AI/LLM and resource-exhaustion guidance, adapt ideas from
Cloudflare's MIT-licensed [security-audit skill](https://github.com/cloudflare/security-audit-skill),
rewritten for WordPress.

## License

[MIT](LICENSE)

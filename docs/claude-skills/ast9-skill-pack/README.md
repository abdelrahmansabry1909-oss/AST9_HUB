# AST9 Claude Skills Pack — canonical tracked copy

This folder is the **durable, version-controlled source of truth** for the AST9 project-specific
Claude guard skills. The skills run from `.claude/skills/`, but that path is gitignored
(see the repo `.gitignore` — `.claude/`, `.agents/`, and `skills-lock.json` are all ignored),
so the runtime copy is local-only and would be lost if the worktree is removed. This tracked
copy under `docs/` preserves them and lets anyone reinstall or review them.

Parent workflow doc: [`AI_WORKFLOW_GUARDRAILS.md`](../../../AI_WORKFLOW_GUARDRAILS.md). These ten
skills are the project-specific extension of those mandatory guardrails.

## 1. What the AST9 skills pack is

A set of ten project-authored Claude skills that encode the **real regressions and process
mistakes already hit during AST9/NeuCore development**, and force the correct checklist before
similar changes are made again. Each skill is a directory containing a `SKILL.md` with YAML
frontmatter (`name`, `description`) whose `description` is a trigger paragraph that lets Claude
Code auto-activate the skill on relevant work.

These are **separate** from the general-purpose marketplace guard skills already installed
(`clean-code-guard`, `docs-guard`, `test-guard`, `woo-guard`, `wp-guard`), which were installed
from `amElnagdy/guard-skills` and are tracked by `skills-lock.json`. The AST9 pack is
project-specific and authored in-repo.

## 2. The ten skills

| Skill | Scope |
|---|---|
| `ast9-decision-journal` | **Cross-cutting.** Forces the reasoning behind an action to be stated *before* it, and corrected in `docs/DECISION_JOURNAL.md` after it is proven wrong. Governs reasoning; the others govern mechanics. |
| `ast9-auth-routing-guard` | Auth/session/client-login routing, the client subscription gate, the `SIGNED_OUT` listener, boot routing, cache-bust tokens. |
| `ast9-agent-boundary-git-guard` | Git/branch/PR/merge flow + Claude(audit), Codex(backend), and Antigravity(frontend) ownership boundaries. |
| `ast9-frontend-launch-guard` | Landing page, mobile nav, Sign-In reachability, public/auth entry visibility. |
| `ast9-realtime-smoke-guard` | Realtime / no-refresh behavior and how to test the *subscribed* state honestly. |
| `ast9-cleanup-archive-guard` | File/folder deletion, archiving, and `git worktree` retirement safety. |
| `ast9-production-verification-guard` | Post-merge/deploy verification and honest smoke labeling (no overclaiming). |
| `ast9-frontend-delivery-guard` | Writing and reporting frontend changes: prove every identifier exists, edit minimally, test in a browser, and never label an unrun check as passed. |
| `ast9-supabase-backend-guard` | All Supabase backend work: migrations, RLS policies, RPCs, `SECURITY DEFINER` functions, triggers, `pg_cron`, and applying anything to the production database. |
| `ast9-payments-webhook-guard` | Payments, billing activation, and provider webhooks (Paymob primary, provider-neutral): HMAC verification, server-side price resolution, idempotency, and the `payment_events` ledger. |

## 3. Which AST9 failure each skill prevents

- **`ast9-decision-journal`** — the recurring shape behind most of the rest: a confident,
  well-measured claim resting on the wrong evidence. Eight logged cases in
  [`docs/DECISION_JOURNAL.md`](../../DECISION_JOURNAL.md), including a grep for `supabase/client`
  that could never match `./client.js` (understating a deletion from 1 file to 4), a sidebar
  quoted as 248px that was really a 64px collapsing rail, a card silently on UA-default margins
  that survived three clean measurement passes, and a 24px prediction sitting inside its own
  ±50px error bar. Enforces: state the evidence *method* and its error bar before acting, name
  the blast radius, say what would falsify the claim and whether you can run it — and write the
  correction into the journal, the PR body and memory rather than only into chat.

- **`ast9-auth-routing-guard`** — the client-login **landing-bounce** bug (PR #53): the subscription
  gate's `sb.auth.signOut()` fired `SIGNED_OUT`, whose global listener redirected to `index.html`,
  so a gated client landed on the marketing page instead of the subscription-inactive screen; the
  subscription read also failed **closed** to `none` and a 4-second boot timeout raced a slower
  `Auth.init()`. Enforces: gate-signout suppression, an `unknown` sentinel (not `none`), a boot
  timeout that does not bounce, unchanged owner/coach routing + owner-only admin, and cache-bust bumps.
- **`ast9-agent-boundary-git-guard`** — direct-push-to-main, cross-agent file overwrites, accidental
  staging of tooling/`dist`/`node_modules`/screenshots/secrets, and wrong-folder edits.
- **`ast9-frontend-launch-guard`** — the **mobile launch blocker** (PR #52): at ≤900px the landing hid
  the Sign-In button behind a hamburger that had no handler and no menu, leaving no path to login.
  Also enforces the CSS-reachability (Antigravity) vs post-login-bounce (Claude) classification.
- **`ast9-realtime-smoke-guard`** — the **false "realtime broken"** conclusion caused by testing a view
  whose channel was never subscribed (the thread was never opened), plus duplicate-channel and
  over-scoped-load mistakes.
- **`ast9-cleanup-archive-guard`** — destroying unpushed/local-only work during cleanup; manual deletion
  of `.claude` worktrees; unapproved `git clean` / `reset --hard`; the `D:\ASThub-current` (current) vs
  `D:\ASThub` (legacy) mix-up.
- **`ast9-production-verification-guard`** — calling a change "production-safe" from a build-only check,
  verifying before the GitHub Pages deploy finished, or trusting the repo artifact instead of the live asset.
- **`ast9-frontend-delivery-guard`** — the 2026-08-02 payments/clients UI rounds, where every defect passed
  `test:unit` **and** `build`: an undeclared `_allClients` that threw `ReferenceError` on every call and
  regressed the **working** admin delete while being reported as "PASS (Code Verified)"; a `#billing-catalog`
  scroll target that never existed; a `PACKAGE_EXPIRED` code the server never sends; a `padding:16px`
  silently dropped while rewriting a style block; a panel docked 35% below the fold at 1366×768 taking the
  Save button off-screen; and the same change reported as done in two consecutive rounds while absent from
  the diff both times. Enforces: grep every identifier before referencing it, edit minimally, run the code
  in a browser, and never write "PASS (Code Verified)".
- **`ast9-supabase-backend-guard`** — frontend-only permission checks (a section hidden in JS while the
  table stays readable by any authenticated user — the frontend is UX, **RLS is the authorization
  boundary**); `SECURITY DEFINER` privilege escalation via a mutable `search_path` or a grant to
  `anon`/`authenticated` where service-role-only was meant; non-idempotent money/period operations that
  double-apply on a retried call; destructive migrations run on prod without approval or backup;
  weakening the owner-only admin model; and leaving test rows or printing real emails/PII/health data
  during a smoke.
- **`ast9-payments-webhook-guard`** — granting paid access from a client-side "success" redirect
  (trivially spoofable by editing the URL); processing an unverified webhook payload; trusting
  client-supplied amount/currency/plan (pay 1 EGP, get the annual plan); replaying or double-processing
  a retried webhook; leaking the HMAC secret or API keys into the repo, logs, or chat; storing card
  data or raw PII in the ledger; and float rounding drift on money.

## 4. Active runtime location

Claude Code discovers and auto-activates project skills from:

```
.claude/skills/<skill-name>/SKILL.md
```

Skills in that path register automatically on load. **That path is gitignored**, so it is the
*runtime* copy, not the canonical source.

Be aware the runtime copies drift and can be **split across checkouts**. As of 2026-11-04 no single
`.claude/skills/` held all ten: the main checkout had eight (missing `ast9-supabase-backend-guard`
and `ast9-payments-webhook-guard`) while a worktree had nine (missing `ast9-decision-journal` and
`ast9-frontend-delivery-guard`). This `docs/` copy is the only complete set — reinstall from it
(§6) rather than copying between runtime locations.

When comparing a runtime copy against this one, **normalize line endings first**. `core.autocrlf`
is `true`, so git stores LF while the working files are CRLF; a naive `diff` reports every line as
changed and makes an identical file look stale. Use `diff --strip-trailing-cr`.

## 5. This folder is the durable canonical copy

```
docs/claude-skills/ast9-skill-pack/<skill-name>/SKILL.md
```

This `docs/` tree **is** tracked by git, so it survives worktree removal, can be reviewed in PRs, and
can be re-installed at any time. Keep it in sync with `.claude/skills/` whenever a skill changes
(edit one, copy to the other).

## 6. Reinstall / copy into the runtime location

From the repo root, mirror the canonical copy into the active runtime path:

```bash
for s in docs/claude-skills/ast9-skill-pack/ast9-*/; do
  name=$(basename "$s")
  mkdir -p ".claude/skills/$name"
  cp "$s/SKILL.md" ".claude/skills/$name/SKILL.md"
done
```

(To go the other direction — refresh this canonical copy from a runtime edit — swap source and
destination.) No package manager is required for these to work; `.claude/skills/` is read directly.
If the team later prefers managed installs, these can instead be published to a git repo and added
via the same skills manager that produced `skills-lock.json`.

## 7. Installing skills must not touch production app code

Copying skill files only ever writes under `.claude/skills/` (runtime) and
`docs/claude-skills/ast9-skill-pack/` (canonical). **Never** edit `app.html`, `index.html`, `js/*`,
`css/*`, `supabase/*`, `package.json`, `.github/workflows/*`, or `.env*` as part of installing,
copying, or updating skills. Skills are instructions for the agent — not application code.

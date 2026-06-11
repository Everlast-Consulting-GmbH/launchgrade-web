# AGENTS.md

Constitution for any AI agent working in this project — Claude Code, Codex, Cursor, or others. This file and `CLAUDE.md` are identical (symlink).

The technical web baseline lives in `./web-standards/AGENTS.md` (snapshot of the Launchgrade Web Standards 2026, versioned per release). That document is intentionally in German because it references German/EU legal frameworks (BFSG, DDG, TDDDG, MStV, EU rulings).

## What this template needs to run

**Nothing.** The template itself is static files (`public/*`), an asset drop zone (`assets/`), docs (`AGENTS.md`, `web-standards/`), and Claude Code skills. There is no build step, no install step, no runtime. Phase 1 produces a single HTML file that opens in any browser.

Runtime dependencies only appear if the user later chooses a stack migration in phase 2 — and the agent handles them then, not upfront.

## Bootstrap (read this first on a fresh clone)

When invoked in a fresh template clone (or after the user clicks GitHub's "Use this template"), follow these steps in order:

1. **Detect context.** Is this a fresh clone? Check for absence of `index.html` and `project.config.json` at root.
2. **Do nothing eagerly.** Do not install Node, do not run `npm install`, do not install global CLIs. Phase 1 needs only a browser.
3. **Trigger `/launchgrade-design`.** That skill captures company info, references, and the `assets/` folder completely (hard gate before generation), shows 3 style directions, then generates the single-HTML draft (`index.html`) via the versioned master prompt. It also writes `project.config.json`.
4. **Install on demand, not upfront.** Tooling appears only when a later user choice requires it:
   - Picks a stack migration in `/launchgrade-setup` (Astro / Next / SvelteKit / Nuxt) → Node 20+ required for scaffold. If no Node, detect platform and recommend the simplest installer (`fnm`, `volta`, or system installer — not `nvm` as a hard requirement).
   - Wants local Lighthouse audit → use `npx lighthouse` (no global install needed).
   - BFSG-relevant + wants local a11y CLI → `npx @axe-core/cli`.
   - Otherwise audit runs against the live URL via PageSpeed Insights API + Mozilla Observatory (both web-based, no install).

## Workflow (three phases, three skills)

```
1. /launchgrade-design   →  capture material completely (hard gate) → 3 style
                            directions (user picks) → single-HTML draft
                            (index.html) via versioned master prompt
                            + project.config.json
2. /launchgrade-setup    →  optional "bring into shape": stack choice (plain
                            HTML default, or migration to Astro/Next/SvelteKit/
                            Nuxt with visual parity check), required files,
                            CSP/headers, head metas, JSON-LD
3. /launchgrade-audit    →  before every release (PageSpeed Insights + Mozilla
                            Observatory; Lighthouse CLI optional)
```

Two rules hold the workflow together:

- **The single HTML is the design truth.** There is no DESIGN.md or COPY.md — the `:root` token block and the comment header inside `index.html` carry brand DNA, palette, and typography.
- **Setup never changes content or look** (design fidelity). Design/copy changes always go back through `/launchgrade-design`.

## When does what trigger

- **New website, draft, design, copy, "looks generic", brand refactor** → `launchgrade-design`
- **"Bring into shape", go live, stack choice/migration, robots.txt, CSP, manifest, security.txt, favicons, deploy** → `launchgrade-setup`
- **URL given + "audit" / "Lighthouse" / "pre-launch"** → `launchgrade-audit`

## Required files in the repo (stack-agnostic)

Already in the template with `{{PLACEHOLDER}}` tokens — the setup skill fills them:

- `public/robots.txt` with AI-crawler defaults (GPTBot, ClaudeBot, Google-Extended, PerplexityBot, Applebot-Extended)
- `public/sitemap.xml`
- `public/llms.txt` for AI discoverability
- `public/.well-known/security.txt` with `Expires` 12 months out
- `public/site.webmanifest`
- `public/404.html`, `public/500.html`
- `SECURITY.md`

Also in the template:

- `assets/` — drop zone for brand material (logo, photos, fonts); inventoried by the design skill before generation
- `.claude/skills/launchgrade-design/master-prompt.md` — versioned quality prompt for single-HTML generation

Not in the template (stack- or asset-specific, comes via skill or manually):

- Favicons (SVG + 192/512/180 PNG) — brand asset, per project
- CSP / security headers (`vercel.json`, `next.config.js`, nginx, Cloudflare)
- JSON-LD Organization on the homepage

## Build / test / audit

Plain HTML (default): no build, no npm — open `index.html` (or serve `public/` after setup) in a browser.

If the user picked a stack migration, the stack adds its own scripts:

```bash
npm run dev          # local dev
npm run build        # production build
npm test             # if tests exist
```

For audit, trigger `/launchgrade-audit <URL>` directly in Claude Code / Codex. No npm wiring — the skill is context-aware.

## Git workflow

- **Branches**: `feat/<topic>`, `fix/<topic>`, `chore/<topic>`, `docs/<topic>`
- **Commits**: Conventional Commits as a recommendation (`feat:`, `fix:`, `refactor:`, `docs:`, `chore:`, `perf:`) — no hook enforces this, but changelog generation and PR review benefit from it
- **Solo repos**: direct push to `main` is fine
- **Multi-contributor repos**: enable GitHub Branch Protection on `main` (Settings → Branches → Add rule: require PR + status checks)

## Code standards (when a JS stack is picked)

- Prefer TypeScript strict mode
- Early returns over deep nesting
- Input validation at system boundaries (Zod / TypeBox / valibot)
- At minimum happy path + 1 edge case per public function
- No empty catch blocks
- Web standards: honor all MUSTs from `./web-standards/AGENTS.md`

## Security

- Never commit secrets — `.env*` is gitignored
- Before push, spot-check: `git diff --cached | grep -iE "(api[_-]?key|secret|token|password)"`
- For BFSG-relevant projects (B2C shop, banking, booking): run `npx @axe-core/cli` with `--tags wcag22aa` before merge

## Keeping standards current

Standards updates come via the public repo (`git pull` or fresh template clone). Version marker lives in `web-standards/.snapshot-version`.

The maintainer-only script `scripts/update-standards.sh` is pure bash, requires no Node, and is not relevant for external users.

## Response style (for AI agents)

- The standards documentation in `web-standards/` is in German. When answering questions about it, respond in the language the user used. Code / headers / commits / tokens stay in English.
- Concise, direct, structured — plan first, then execute
- Em-dashes in user-facing copy: use sparingly — maximum one per longer paragraph
- For BFSG relevance: flag proactively
- For GDPR violations (Google Fonts CDN, hardcoded reCAPTCHA, US tools without DPA): mark clearly

## Anti-patterns

- ❌ Generating the draft before the material capture is confirmed complete — the hard gate exists because staged input produces mismatched results
- ❌ Creating DESIGN.md or COPY.md — the single HTML is the design truth
- ❌ Setup changing content, copy, or look — design fidelity rule
- ❌ Installing Node / nvm / npm packages on first invocation "just in case" — tooling appears only on stack migration
- ❌ Reciting standards from memory instead of reading `./web-standards/AGENTS.md`
- ❌ Skipping the master prompt when generating the draft
- ❌ Generic `robots.txt` without AI-crawler configuration
- ❌ CSP with `unsafe-inline` without documented justification
- ❌ Embedding Google Fonts via `fonts.googleapis.com`
- ❌ Multiple `<h1>` per page
- ❌ Images without `width`/`height` attributes
- ❌ `security.txt` without `Expires` or with `Expires < today`

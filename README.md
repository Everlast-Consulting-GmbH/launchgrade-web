# Launchgrade — Web Standards 2026

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

Stack-agnostic starter template + technical baseline for modern web projects (2026 edition). Single-HTML-first: the first phase produces a self-contained `index.html` draft as the visual ground truth, before any stack decision. Compliance-grade defaults for EU markets (BFSG / GDPR / TDDDG), Core Web Vitals, security headers, AI-crawler policy, and anti-slop design hygiene. Operationalized as three Claude Code skills.

> The standards documentation in `web-standards/` is intentionally in German — it cites German/EU legal frameworks (BFSG, DDG, TDDDG, MStV, ECJ rulings) where translation would lose legal precision. Tech terms (WCAG, CWV, CSP, JSON-LD) are English throughout, so most of the document is readable without German.

## How to use

You drive this template through an AI agent (Claude Code, Codex, Cursor). The template itself installs nothing.

1. Click **"Use this template"** on this repo → "Create a new repository"
2. Open the new repo in **Claude Code** (or Codex / Cursor)
3. Drop your brand material (logo, photos, fonts) into `assets/`
4. Say: **"design"**

The agent captures your company info, references, and assets completely (hard gate — generation only starts when the material is confirmed complete), shows 3 style directions, and generates a self-contained `index.html` draft via a versioned master prompt. When you approve the draft, say **"setup"** to bring it into production shape, and **"audit"** before release.

## What you actually need installed

- **A browser**
- **An AI agent** — [Claude Code](https://docs.claude.com/en/docs/claude-code), Codex, or Cursor
- **Git** — or just use the GitHub template button + GitHub web editor

That's it. No Node, no nvm, no global CLIs. The draft is a single HTML file that opens in any browser. Node only appears if you later choose a stack migration (Astro / Next.js / SvelteKit / Nuxt) or want the local smoke test / Lighthouse run, installed on demand by the agent.

## What's inside

- **Launchgrade Web Standards 2026** (`web-standards/AGENTS.md` + `checklist.md`) — versioned baseline covering Performance (Core Web Vitals 2026), Accessibility (WCAG 2.2 AA + BFSG), SEO (classic + AI-crawler era), Security (CSP, headers, cookies, auth), Privacy (GDPR / TDDDG), Motion, PWA. RFC-2119 levels (MUST / Conditional MUST / SHOULD / MAY).
- **Three Claude Code skills** (`.claude/skills/`) — Draft → Shape → Audit:
  - `launchgrade-design` — phase 1: captures material with a hard gate, 3 style directions, generates the single-HTML draft via the versioned master prompt (`master-prompt.md`). The HTML is the design truth — no DESIGN.md.
  - `launchgrade-setup` — phase 2 (optional): brings the approved draft into production shape — stack choice (plain HTML default, or migration with visual parity check), required files, CSP, head metas, JSON-LD. Never changes content or look.
  - `launchgrade-audit` — phase 3: pre-/post-launch via PageSpeed Insights + Mozilla Observatory (web-based, zero install); Lighthouse CLI optional
- **Asset drop zone** (`assets/`) — logo, photos, fonts; inventoried before generation
- **Required files** in `public/` with `{{PLACEHOLDER}}` tokens (robots.txt, sitemap.xml, llms.txt, .well-known/security.txt, site.webmanifest, 404/500)
- **Claude Code settings** (`.claude/settings.json`) — permission allowlist for audit tooling, deny rules against force push and publish

## Workflow

```
1. /launchgrade-design   →  material capture (hard gate) → 3 style directions
                            → single-HTML draft (index.html) + project.config.json
2. /launchgrade-setup    →  optional: stack choice (plain HTML default, or
                            migration), required files, CSP, metas, JSON-LD
3. /launchgrade-audit    →  pre-launch (PageSpeed Insights + Observatory),
                            findings as Blockers / Recommended / Nice-to-have
```

## Reading the standards directly

- **[`web-standards/AGENTS.md`](web-standards/AGENTS.md)** — full standards with §-structure and pass thresholds
- **[`web-standards/checklist.md`](web-standards/checklist.md)** — compact pre-launch checklist

These files are self-contained — you don't need the template or the skills to use them as a reference. Also usable as a contract appendix ("Conformance to Launchgrade Web Standards 2026").

## What's NOT in the template

- Framework-specific code (Next.js, Astro, SvelteKit) — the setup skill scaffolds a stack on demand, only if you choose migration
- `index.html` / `project.config.json` — per project via `launchgrade-design`
- Brand favicons — a placeholder `public/favicon.svg` ships with the template; brand SVG + PNGs (192/512/180) come per project
- CI/CD setup — intentionally omitted for solo / small-team repos
- `package.json` at root — there's nothing to install; a migrated stack adds its own

## Manual path (no agent)

If you really want to work without an agent: clone the repo, build your own `index.html`, edit the `{{PLACEHOLDER}}` tokens in `public/*` and `SECURITY.md` by hand, follow `web-standards/checklist.md`, deploy. The standards work as a contract reference even without the skill workflow.

## Stack choice

See [`docs/STACK_CHOICE.md`](docs/STACK_CHOICE.md) — decision matrix for marketing site / blog / shop / SaaS marketing. The stack decision happens in phase 2 (`launchgrade-setup`), after the single-HTML draft exists. Plain HTML is the default.

## Contributing

Issues and pull requests welcome. For larger changes, please open an issue first. See [`CONTRIBUTING.md`](CONTRIBUTING.md) for branch conventions and commit style.

## Maintenance

Best-effort maintenance. No SLA, no warranty — see [`LICENSE`](LICENSE). Standards updates follow the changelog in `web-standards/AGENTS.md`.

> The script `scripts/update-standards.sh` is a maintainer-only bash tool for syncing from a private standards source repo and is not relevant for external users — the snapshot in `web-standards/` is already self-contained.

## License

[MIT](LICENSE) © Everlast Consulting GmbH

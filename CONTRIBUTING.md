# Contributing

This is proprietary software (see [`LICENSE`](LICENSE)). These conventions apply to Everlast Consulting GmbH and authorized contributors, not to the general public.

Lean conventions for solo / small-team repos — no enforcement layer, no CI, AI handles format and style.

## Branches

- `main` — production
- `feat/<topic>` — new features
- `fix/<topic>` — bug fixes
- `chore/<topic>` — maintenance

If a project grows to multi-contributor, enable **GitHub Branch Protection** on `main` (Settings → Branches → Add rule: require PR, require status checks). For solo repos this is not required — direct push to `main` is fine.

## Commits

[Conventional Commits](https://www.conventionalcommits.org/) as a recommendation:

```
feat: add cookie consent banner
fix: correct hreflang on de/en switch
refactor: extract header into component
docs: update CSP example
chore(deps): bump astro to 5.4
perf: lazy-load below-fold images
```

No hook enforces this — use it as a convention because changelog generation and PR review benefit from it.

## Standards

All changes must respect the Launchgrade Web Standards — `./web-standards/AGENTS.md` (snapshot in this repo).

For BFSG-relevant projects, additionally run `@axe-core/cli` before merging to `main`.

## Issues & Pull Requests

- Use GitHub Issues for bug reports, feature requests, or to discuss larger changes before opening a PR.
- Keep PRs focused — one logical change per PR. Multiple unrelated fixes in one PR are harder to review.
- Reference any related Issue in the PR description.
- For standards changes (`web-standards/AGENTS.md` or `checklist.md`), add a Changelog entry at the bottom of the file with the date and version bump rationale.

## Git settings (once per project)

Recommended but optional:

```bash
git config commit.gpgsign true       # Signed commits (if GPG is set up)
git config pull.rebase true          # rebase instead of merge on pull
git config core.autocrlf input       # Keep LF, convert CRLF on input
```

---
name: launchgrade-seo
description: Lightweight SEO strategy and content workflow for Launchgrade projects. Use when working on keywords, search intent, SEO strategy, content roadmaps, page maps, content briefs, blog/article planning, commercial landing pages, or Google Search Console exports. Creates or updates the small `seo/` workspace without API integrations, scraping, rank tracking, or paid-tool assumptions.
---

# Launchgrade SEO Skill

Optional companion to the Launchgrade workflow. This skill covers the content/keyword side of SEO while `launchgrade-setup` and `launchgrade-audit` keep owning technical SEO.

## Scope

Do:

- Map business goals to keywords and pages.
- Define search intent before writing.
- Create concise page or article briefs.
- Review existing pages for keyword focus, intent fit, internal links, CTA fit, and cannibalization risk.
- Interpret Google Search Console CSV exports when the user provides them.

Do not:

- Add API integrations, OAuth, SERP scraping, rank tracking, or paid-tool dependencies.
- Claim reliable search volume or keyword difficulty without a provided source.
- Generate bulk SEO pages without distinct search intent and human-review notes.
- Modify technical SEO rules owned by `launchgrade-setup` or `launchgrade-audit`.

## Files

Use these repo files:

- `seo/README.md` for the 80/20 process.
- `seo/strategy.md` for business goal, keyword map, page map, and roadmap.
- `seo/brief-template.md` as the source template for page and blog briefs.

If the files are missing, create them from the template defaults. Keep them lightweight and editable by non-technical users.

## Workflow

1. **Clarify goal**: lead type, target audience, offer, geography/language, and top business priorities.
2. **Collect keyword inputs**: user-provided seed keywords, competitor URLs, free-tool results, SERP observations, or Search Console CSVs. If no data is provided, label the output as qualitative research.
3. **Map pages**: assign one primary keyword and one search intent to each indexable page. Avoid two pages serving the same intent unless the distinction is explicit.
4. **Prioritize**: rank pages by business value, intent strength, current coverage, and implementation effort. Do not invent volume numbers.
5. **Create briefs**: for each priority page/article, copy `seo/brief-template.md` into a working brief or inline the same sections in the answer.
6. **QA before publish**: check primary keyword, intent, H1/H2 fit, answer-first sections, internal links, CTA, sources/proof, FAQ opportunities, and human-review notes.
7. **After launch**: use Search Console data to find pages with impressions but weak position/CTR, cannibalized queries, and refresh opportunities.

## Output Rules

- Separate evidence from assumptions.
- Mark unknown volume/difficulty as `unknown`, not guessed.
- Prefer a small roadmap of 5-15 high-value pages over large programmatic lists.
- For regulated topics such as legal, medical, finance, AI Act, GDPR, or security, include explicit human-review notes.
- Keep recommendations actionable: page, keyword, intent, next edit.

## Handoff

After SEO planning:

- For visual/copy changes, use `launchgrade-design`.
- For production metadata/files/headers, use `launchgrade-setup`.
- Before release, use `launchgrade-audit`.

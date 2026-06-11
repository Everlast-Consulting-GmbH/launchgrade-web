# Spec: Single-HTML-first Workflow

Datum: 2026-06-11
Status: freigegeben (Design-Review mit Maintainer abgeschlossen)

## Motivation

Der bisherige Workflow (Setup → Design → Build → Audit) hat zwei Probleme gezeigt:

1. **Diskrepanz-Problem:** Design-Input kam gestaffelt (erst Stilbeschreibung, später
   Ressourcen). Ergebnis: Die generierte Seite passte nicht zum nachgelieferten
   Material. Material muss vollständig erfasst sein, bevor generiert wird.
2. **Qualität war promptabhängig:** Ob eine hochwertige Seite entstand, hing am
   Ad-hoc-Prompt des Users. Die Qualitäts-Constraints (Anti-Slop, A11y, Typo)
   müssen als versionierter Standard-Prompt im Skill liegen, nicht im Kopf.

Außerdem zwang der alte Flow zur Stack-Entscheidung, bevor irgendetwas Sichtbares
existierte. Neu: Zuerst entsteht eine lightweight Single HTML als visuelle
Ground-Truth, die Stack-Frage fällt erst danach (und nur optional).

## Ziel-Workflow

```
1. /launchgrade-design  →  Material komplett erfassen (Gate) → Style-Picker
                           (3 Richtungen) → Single HTML via Master-Prompt
                           → index.html + assets/ + project.config.json
2. /launchgrade-setup   →  optional „in Form bringen": Stack-Wahl (Plain HTML
                           Default, Astro/Next/SvelteKit/Nuxt als Migration),
                           Pflicht-Files, CSP/Headers, <head>-Metas, JSON-LD
3. /launchgrade-audit   →  vor jedem Release (unverändert: Lighthouse,
                           Observatory, PSI, Runtime-Browser-Checks)
```

DESIGN.md und COPY.md entfallen ersatzlos. Die Single HTML ist die Design- und
Copy-Wahrheit (Token-Block + Kommentar-Header übernehmen die Rolle der DESIGN.md).

## Phase 1: `launchgrade-design` (umgebaut)

### Schritt 1 — Material-Erfassung mit hartem Gate

Der Skill fragt aktiv ab und sammelt, **bevor** irgendetwas generiert wird:

- **Unternehmensinfos:** Brand-Name, Tagline/USP, Branche, Leistungen,
  Kontaktdaten, Tonalität, Sprache
- **Referenzen:** URLs (`WebFetch`), Screenshots/Logo/Moodboards (`Read` multimodal)
- **Assets:** `assets/`-Ordner im Repo-Root wird inventarisiert (Logo, Fotos,
  Fonts). Fehlt Material, wird der User aufgefordert, es jetzt abzulegen.
- **Copy-Rohmaterial:** vorhandene Texte, USPs, FAQ-Stoff

**Gate (AskUserQuestion, Pflicht):** „Material komplett? Generierung startet erst
nach Bestätigung." Nachgeliefertes Material nach der Generierung ist eine bewusste
neue Iteration, kein stilles Reinmischen.

Erfasste Daten landen in `project.config.json` (Schema siehe unten), damit Setup
später nichts doppelt fragt.

### Schritt 2 — Style-Picker (bleibt)

Wie bisher: 3 polare Style-Richtungen mit Named Aesthetics als eine HTML-Datei
unter `.launchgrade/mockups/style-picker.html`, im Browser öffnen, Wahl per
`AskUserQuestion`. Direktions-Heuristik nach Branche bleibt erhalten.

### Schritt 3 — Single HTML via Master-Prompt

Der Qualitäts-Prompt liegt versioniert unter
`.claude/skills/launchgrade-design/master-prompt.md` und wird vom Skill bei jeder
Generierung gelesen (nie aus dem Gedächtnis). Inhalt:

- **Anti-Slop-Constraints:** keine Inter als Default, OKLCH-Palette (keine
  Tailwind-Slate), keine Lila-Pink-Gradients, keine generischen Lucide-Hero-Icons,
  Named Aesthetic Pflicht, asymmetrische Layout-Defaults, echte Type-Scale
- **A11y-Hard-Constraints (nicht verhandelbar):** Skip-Link, semantische
  Landmarks, genau ein `<h1>`, Kontrast-Paare, `:focus-visible`,
  Target-Size ≥ 24×24 px, `<html lang>` gesetzt, `prefers-reduced-motion` defensiv
- **Static-Disziplin:** kein JS-Motion-Layer im Draft (kein IntersectionObserver,
  keine Scroll-Reveals); CSS-only `:hover`/`:focus` erlaubt
- **Self-Contained-Regeln:** CSS inline im `<style>`-Block, Design-Tokens als
  klar markierter `:root`-Block, Bilder relativ aus `assets/`, Fonts self-hosted
  oder System-Stack (nie Google Fonts CDN), kein Build-Step nötig
- **Copy-Regeln:** Voice/Tone aus erfasster Tonalität, fehlende Fakten (Telefon,
  Adressen, Termine) nie erfinden — `{{TODO: …}}`-Marker setzen

### Schritt 4 — Output-Konventionen

- `index.html` im Repo-Root, sofort im Browser zu öffnen
- Kommentar-Header am Dateianfang: Brand-DNA (ist/ist-nicht-Adjektive), Named
  Aesthetic, gewählte Style-Richtung — die „neue DESIGN.md" in Kurzform
- `:root`-Token-Block (Farben OKLCH, Typo, Spacing) als Single Source of Truth
- `assets/` referenziert, nichts base64-inlinen (Dateigröße)

### Schritt 5 — Iteration + Hard Gate

- Nach jedem Build: HTML im Browser öffnen (Tool-Detection wie bisher:
  `agent-browser` → Chrome MCP → Fallback Pfad-Ausgabe)
- User verfeinert im Dialog; Skill iteriert auf der `index.html`
- **Hard Gate (AskUserQuestion, Pflicht):** „Draft freigegeben → weiter zu
  `/launchgrade-setup`?" Kein automatischer Übergang. Offene `{{TODO: …}}`-Marker
  werden beim Gate gelistet.

## Phase 2: `launchgrade-setup` (umgebaut)

### Voraussetzungs-Check

`index.html` + `project.config.json` müssen existieren. Wenn nicht: Hinweis
„erst `/launchgrade-design` ausführen", kein Blind-Scaffold auf leerem Repo.

### Technische Erfassung (nur was Design nicht erfasst hat)

Domain, hreflang bei Mehrsprachigkeit, BFSG-Relevanz (B2C-Shop / Banking /
Buchung), Deploy-Ziel (Vercel / Cloudflare / nginx / Apache). Brand-Daten kommen
aus `project.config.json` — nichts wird doppelt gefragt.

### Stack-Wahl (jetzt hier, mit Draft als Ground-Truth)

- **Plain HTML (Default):** vorhandene `index.html` wird produktiv gemacht —
  `<head>` vervollständigt (canonical, OG, Twitter Card, theme-color, JSON-LD
  Organization + WebSite), Pflicht-Files-Platzhalter gefüllt, CSP/Headers je
  Deploy-Ziel, Favicon-Set.
- **Migration (Astro / Next / SvelteKit / Nuxt):** Scaffold non-interactive wie
  bisher (npm, TS strict). Danach: HTML in Components zerlegen (Section = Component),
  `:root`-Token-Block → `src/styles/tokens.css` (bzw. Stack-Pendant), Draft als
  `.launchgrade/draft.html` archiviert. **Pflicht: visueller Abgleich** Draft vs.
  migrierte Seite im Browser, bevor Setup „fertig" meldet.

### Neue Grundregel: Design-Treue

Setup darf Inhalt und Optik **nicht verändern**, nur technisch ergänzen. Ersetzt
die alte Regel „Setup ist Foundation, kein Content" (die existierte nur, weil
Setup früher zuerst lief). Die Placeholder-Welcome-Page-Logik entfällt.

### Bleibt

Smoke-Test (DOM-Checks: title, theme-color, lang, ein h1; Pflicht-Files via curl
inkl. Inhalts-Checks), Pflicht-Schritt AGENTS.md-Kapitel lesen, DSGVO-/BFSG-Flags.

## Phase 3: `launchgrade-audit` (kleine Anpassungen)

Inhaltlich unverändert (Lighthouse, Observatory, PSI, Runtime-Verifikation §4a–4c).
Anpassungen:

- Phasen-Nummerierung/Verweise auf den neuen Workflow
- Trigger-Abgrenzung zu den umgebauten Skills
- §4a-grep-Pfade prüfen zusätzlich Root-`index.html`, nicht nur `src/` (sauberer
  Plain-HTML-Pfad)

## `project.config.json` (Schema)

Geschrieben von Design (Brand-Teil), ergänzt von Setup (Technik-Teil):

```json
{
  "brand_name": "…",
  "brand_tagline": "…",
  "industry": "…",
  "services": ["…"],
  "contact": { "email": "…", "phone": "…", "address": "…" },
  "tonality": "…",
  "lang": "de",
  "primary_color": { "hex": "#…", "oklch": "oklch(… … …)" },
  "style_direction": "Named Aesthetic der gewählten Variante",
  "domain": null,
  "bfsg_relevant": null,
  "stack": null,
  "deploy_target": null,
  "standards_snapshot": "Inhalt von web-standards/.snapshot-version"
}
```

`null`-Felder füllt Setup. Kontaktfelder ohne erfassten Wert bleiben `null`
(nie erfinden).

## Template- und Docs-Änderungen

| Datei | Änderung |
|---|---|
| `AGENTS.md` / `CLAUDE.md` (Root) | Bootstrap: frischer Klon triggert `/launchgrade-design`; Workflow-Diagramm, „When does what trigger", Anti-Patterns neu |
| `README.md` | „How to use" → **„Say: design"**, Workflow-Block, „What's inside" neu |
| `docs/STACK_CHOICE.md` | Hinweis: Stack-Entscheidung fällt in Phase 2 (Setup), nach dem Draft |
| `assets/` (neu) | Ordner mit kurzer README: „Logo, Fotos, Fonts hier ablegen — der Design-Skill bindet sie ein" |
| `.claude/skills/launchgrade-design/master-prompt.md` (neu) | versionierter Qualitäts-Prompt |
| `.claude/skills/launchgrade-design/SKILL.md` | Umbau auf Phase 1 (Material-Gate, Master-Prompt, Single HTML) |
| `.claude/skills/launchgrade-setup/SKILL.md` | Umbau auf Phase 2 (In-Form-Bringer, Stack-Wahl, Design-Treue) |
| `.claude/skills/launchgrade-audit/SKILL.md` | kleine Anpassungen (Verweise, grep-Pfade) |
| `public/*`, `SECURITY.md`, `web-standards/`, `.claude/settings.json` | unverändert |

## Nicht-Ziele

- Keine Änderung an den Web-Standards selbst (`web-standards/` ist Snapshot)
- Keine neuen Skills, kein Merge bestehender Skills
- Keine CI/CD-Einführung
- Audit-Logik (Tools, Schwellen, Buckets) bleibt unangetastet

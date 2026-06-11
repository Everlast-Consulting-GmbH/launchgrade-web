# Single-HTML-first Workflow Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Umbau des Launchgrade-Templates auf den Single-HTML-first Workflow: `/launchgrade-design` wird Phase 1 (Material-Gate → Style-Picker → Single HTML via Master-Prompt), `/launchgrade-setup` wird Phase 2 (In-Form-Bringer mit Stack-Wahl), `/launchgrade-audit` bekommt kleine Anpassungen, alle Docs werden umgestellt.

**Architecture:** Reines Markdown-/Template-Repo ohne Build-Step und ohne Tests — "Tests" sind hier deterministische Verifikations-Greps nach jedem Task (keine veralteten Referenzen, konsistente Pfade). Spec: `docs/superpowers/specs/2026-06-11-single-html-first-workflow-design.md`. Arbeitsbranch: `feat/single-html-first-workflow` (existiert bereits, Spec ist committet).

**Tech Stack:** Markdown, Claude-Code-Skills (SKILL.md mit YAML-Frontmatter), Bash-Verifikation via grep.

**Sprachkonvention:** Skill-Inhalte und Master-Prompt auf Deutsch (wie bisher), Frontmatter-`description` auf Englisch mit deutschen Trigger-Wörtern (wie bisher), Root-`AGENTS.md`/`README.md` auf Englisch (wie bisher). Commits Englisch, Conventional Commits.

---

## File-Struktur (was wird angefasst)

| Datei | Aktion | Verantwortung |
|---|---|---|
| `.gitignore` | Modify | `.launchgrade/` ignorieren |
| `assets/README.md` | Create | Drop-Zone für Brand-Material erklären |
| `.claude/skills/launchgrade-design/master-prompt.md` | Create | versionierter Qualitäts-Prompt |
| `.claude/skills/launchgrade-design/SKILL.md` | Rewrite | Phase 1: Material-Gate, Style-Picker, Single HTML |
| `.claude/skills/launchgrade-setup/SKILL.md` | Rewrite | Phase 2: In-Form-Bringer, Stack-Wahl, Design-Treue |
| `.claude/skills/launchgrade-audit/SKILL.md` | Edit (3 Stellen) | grep-Pfade, Übergabe-Verweis |
| `AGENTS.md` (Root, = `CLAUDE.md` Symlink) | Rewrite | Bootstrap, Workflow, Trigger, Anti-Patterns |
| `README.md` | Rewrite | How to use, Workflow, What's inside |
| `docs/STACK_CHOICE.md` | Edit (2 Stellen) | Stack-Wahl jetzt Phase 2 |

Entscheidung (Implementierungsdetail über Spec hinaus): Im **Plain-HTML-Pfad** verschiebt Setup `index.html` + `assets/` nach `public/` (Publish-Dir = `public/`, dort liegen schon alle Pflicht-Files). Im **Migrations-Pfad** wird der Draft nach `.launchgrade/draft.html` archiviert.

---

### Task 1: `.gitignore` + `assets/`-Ordner

**Files:**
- Modify: `.gitignore`
- Create: `assets/README.md`

- [ ] **Step 1: `.launchgrade/` zu `.gitignore` hinzufügen**

In `.gitignore` nach dem Block `# Audit outputs … audit-reports/` einfügen:

```
# Launchgrade working artifacts (style picker, draft archive)
.launchgrade/
```

- [ ] **Step 2: `assets/README.md` anlegen**

```markdown
# assets/

Brand-Material hier ablegen, bevor `/launchgrade-design` läuft:

- **Logo** (`logo.svg` / `logo.png`)
- **Fotos** (Team, Produkte, Location — Web-tauglich, möglichst ≥ 1600 px Breite)
- **Fonts** (WOFF2, lizenziert für Self-Hosting)
- **Grafiken / Icons** (SVG bevorzugt)

Der Design-Skill inventarisiert diesen Ordner bei der Material-Erfassung und
bindet die Dateien relativ in `index.html` ein (`assets/...`). Nichts wird
base64-inlined.

Hinweise:

- Dateinamen sprechend und in Kleinbuchstaben (`team-buero.jpg`, nicht `IMG_4711.JPG`)
- Keine Google-Fonts-CDN-Verweise — Fonts gehören als Dateien hierher (DSGVO)
```

- [ ] **Step 3: Verifizieren**

Run: `git check-ignore -v .launchgrade/ ; ls assets/`
Expected: `.gitignore:…:.launchgrade/` und `README.md`

- [ ] **Step 4: Commit**

```bash
git add .gitignore assets/README.md
git commit -m "feat(template): add assets/ drop zone and ignore .launchgrade/

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

---

### Task 2: Master-Prompt anlegen

**Files:**
- Create: `.claude/skills/launchgrade-design/master-prompt.md`

- [ ] **Step 1: Datei mit folgendem Inhalt anlegen**

````markdown
# Master-Prompt: Single-HTML-Draft

Versionierter Qualitäts-Prompt für die Generierung des Single-HTML-Entwurfs.
Der Design-Skill liest diese Datei vor JEDER Generierung — nie aus dem
Gedächtnis arbeiten. Änderungen an diesem Prompt sind Template-Releases
(Conventional Commit, Git-History ist die Versionierung).

## Auftrag

Baue eine vollständige, self-contained Single-HTML-Seite (`index.html` im
Repo-Root) auf Basis von:

1. `project.config.json` (Brand-Daten, Tonalität, Sprache)
2. der gewählten Style-Direktion (Named Aesthetic aus dem Style-Picker)
3. dem Inventar des `assets/`-Ordners
4. dem erfassten Copy-Rohmaterial

Alle vier Inputs müssen vorliegen. Fehlt einer: zurück zur Material-Erfassung,
nicht improvisieren.

## Output-Konventionen

- Eine Datei: `index.html`, ohne Build-Step im Browser lauffähig
- Kommentar-Header am Dateianfang:

  ```html
  <!--
    Brand-DNA: <3-5 Adjektive die die Brand IST> | NICHT: <3-5 Adjektive>
    Named Aesthetic: <konkret, z.B. "Editorial Serif nach Mr-Porter-Vorbild">
    Style-Direktion: <Variante A/B/C aus dem Style-Picker>
    Generiert: <Datum> · Master-Prompt: .claude/skills/launchgrade-design/master-prompt.md
  -->
  ```

- CSS komplett inline in einem `<style>`-Block im `<head>`
- Design-Tokens als klar markierter `:root`-Block am Anfang des CSS — Single
  Source of Truth für Farbe, Typo, Spacing:

  ```css
  :root {
    /* COLOR — OKLCH, kalibriert, keine Tailwind-Defaults */
    --color-primary: oklch(0.55 0.15 250);
    --color-surface: oklch(0.98 0.005 250);
    --color-text: oklch(0.22 0.02 250);
    /* TYPE SCALE — echte Werte, kein Default-Stack */
    --font-display: "<Display-Font>", serif;
    --font-body: "<Body-Font>", system-ui, sans-serif;
    --step-0: clamp(1rem, 0.95rem + 0.25vw, 1.125rem);
    --step-3: clamp(2rem, 1.6rem + 2vw, 3.25rem);
    /* SPACING */
    --space-m: clamp(1.5rem, 1.2rem + 1.5vw, 2.5rem);
  }
  ```

  (Werte sind Beispiele — konkrete Werte kommen aus der gewählten
  Style-Direktion.)

- `<head>`-Minimum: charset, viewport, `<title>`, `<meta name="description">`,
  `<meta name="color-scheme">` — canonical/OG/JSON-LD ergänzt später Setup
- Bilder relativ aus `assets/` (`<img src="assets/...">`), immer mit
  `width`/`height`-Attributen und sinnvollem `alt`. Nichts base64-inlinen.
- Fonts: self-hosted via `@font-face` aus `assets/` ODER System-Stack.
  NIE `fonts.googleapis.com`.

## Anti-Slop (hart)

- KEIN Inter/Roboto/Open Sans als Default — Schriftwahl folgt der Named Aesthetic
- KEINE Lila-Pink-Gradients, kein Glass-Morphism ohne Brand-Argument
- KEINE Tailwind-Slate-Palette — Farben in OKLCH, aus der Style-Direktion kalibriert
- KEINE generischen Icon-Floskeln im Hero (Lucide-Default-Icons o.ä.)
- KEIN "Get Started"-Default-CTA — CTA-Text aus USP/Copy-Material ableiten
- Asymmetrische Layout-Defaults — kein 3-Spalten-Bento als Reflex
- Echte Type-Scale: Display-Headline deutlich größer als Body (≥ 2.5×),
  kein Einheitsbrei

## Accessibility (nicht verhandelbar — bei Konflikt gewinnt A11y über Brand)

- `<html lang="…">` aus `project.config.json`
- Skip-Link `<a href="#main">` als erstes Body-Element
- Landmarks: `<header>`, `<nav>`, `<main id="main">`, `<footer>`
- Genau EIN `<h1>`, logische Heading-Hierarchie ohne Sprünge
- Kontrast: Fließtext ≥ 4.5:1, Large Text / UI ≥ 3:1 — Token-Paare entsprechend wählen
- `:focus-visible`-Indicator für alle interaktiven Elemente
- Target-Size ≥ 24×24 px
- `prefers-reduced-motion` defensiv respektieren (auch ohne Animationen)

## Static-Disziplin (der Draft ist Phase 1 Static)

- KEIN JavaScript-Motion-Layer: kein IntersectionObserver, keine Scroll-Reveals,
  keine Spring-/Tween-Animations, keine View Transitions
- CSS-only `:hover` / `:focus-visible`-Styles sind erlaubt
- JS überhaupt nur, wenn funktional zwingend (z.B. Mobile-Nav-Toggle) — dann
  minimal, inline, ohne Dependencies

## Copy-Regeln

- Voice/Tone aus erfasster Tonalität, Sprache aus `project.config.json`
- Fakten NUR aus dem erfassten Material — Telefon, Adressen, Preise, Termine,
  Zertifikate NIE erfinden. Fehlt ein Fakt: `{{TODO: <was fehlt>}}` als
  sichtbarer Inline-Marker
- Em-dashes sparsam (maximal einer pro längerem Absatz)
- Microcopy (Buttons, Formulare, Fehlertexte) konkret statt generisch

## Struktur-Heuristik

Sektionen aus dem Copy-Material ableiten — typisch: Hero (USP + primärer CTA),
Leistungen/Angebot, Über uns/Trust, Social Proof (nur mit echtem Material),
FAQ (nur mit echtem Material), Kontakt/Footer (inkl. Impressum-/Datenschutz-Links
als Platzhalter-Anker `#impressum` / `#datenschutz`). Keine Sektion erfinden,
für die kein Material existiert.

## Self-Check vor Abgabe (alle Punkte prüfen, bei Fail nachbessern)

1. Datei öffnet ohne Build-Step im Browser, keine 404 auf `assets/`-Referenzen
2. Genau ein `<h1>`; Landmarks vollständig; Skip-Link vorhanden
3. Kein `fonts.googleapis.com`, kein CDN-Script
4. Alle Bilder mit `width`/`height`/`alt`
5. Token-Block vollständig — keine hardcoded Farben außerhalb von `:root`
6. Kein erfundener Fakt — alle Lücken als `{{TODO: …}}` markiert
7. Kommentar-Header vollständig
````

- [ ] **Step 2: Verifizieren**

Run: `grep -c "TODO:" .claude/skills/launchgrade-design/master-prompt.md && grep -n "fonts.googleapis" .claude/skills/launchgrade-design/master-prompt.md`
Expected: TODO-Treffer nur als `{{TODO: …}}`-Konvention (Marker-Beschreibung), `fonts.googleapis` nur in Verbots-Sätzen.

- [ ] **Step 3: Commit**

```bash
git add .claude/skills/launchgrade-design/master-prompt.md
git commit -m "feat(design-skill): add versioned master prompt for single-HTML drafts

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

---

### Task 3: `launchgrade-design` SKILL.md neu schreiben

**Files:**
- Rewrite: `.claude/skills/launchgrade-design/SKILL.md`

- [ ] **Step 1: Datei komplett ersetzen mit folgendem Inhalt**

````markdown
---
name: launchgrade-design
description: Phase 1 of the Launchgrade workflow — builds a lightweight, self-contained single-HTML draft as the visual ground truth of the site. Captures company info, references, and assets completely upfront (hard gate before generation), shows 3 style directions (user picks one), then generates index.html via the versioned master prompt (anti-slop, a11y hard constraints, static discipline). Writes project.config.json. No DESIGN.md/COPY.md — the HTML is the design truth. Triggers on "new website", "landing page", "design", "draft", "first draft", "look", "brand", "single HTML", "copy", "looks generic", "style guide", "neue Website", "Webseite erstellen", "Landing Page", "Entwurf", "erster Entwurf", "Design", "Anti-Slop", "sieht generisch aus", "Webseiten-Text".
---

# Launchgrade Web Design Skill

Erste Phase im Launchgrade-Workflow: **Draft**. Baut eine lightweight Single
HTML als visuelle Ground-Truth. Danach optional: `/launchgrade-setup` (in Form
bringen, Stack-Wahl) und `/launchgrade-audit` (Verifikation vor Release).

## Grundsatz

**Material zuerst, Generierung danach.** Die häufigste Fehlerquelle: Input
kommt gestaffelt (erst Stilwunsch, später Ressourcen) und das Ergebnis passt
nicht zum Material. Deshalb hartes Gate nach der Material-Erfassung —
generiert wird erst, wenn der User „komplett" bestätigt. Nachgeliefertes
Material ist eine bewusste neue Iteration, kein stilles Reinmischen.

Output des Skills:

- `index.html` im Repo-Root — self-contained, ohne Build-Step im Browser lauffähig
- `project.config.json` — erfasste Brand-Daten (Setup fragt nichts doppelt)

DESIGN.md und COPY.md existieren nicht mehr. Der `:root`-Token-Block und der
Kommentar-Header in der HTML übernehmen ihre Rolle.

## Wann triggern

- Neues Web-Projekt / frischer Template-Klon (erster Skill im Workflow)
- „Webseite erstellen", „Entwurf", „Design", Re-Design, „sieht generisch aus"
- Brand-/Style-Fragen, neue Style-Richtung für einen bestehenden Draft
- Copy-/Text-Arbeit am Draft

Nicht triggern bei: Pflicht-Files / CSP / Stack / Deploy (→ `launchgrade-setup`),
Audit / Pre-Launch-Check (→ `launchgrade-audit`), Backend-/Admin-Tasks.

## Vorgehen

1. **Material-Erfassung — vollständig, bevor irgendetwas generiert wird:**

   a) **Unternehmensinfos** abfragen (kompakt, in einem Schwung): Brand-Name,
      Tagline/USP (1 Satz), Branche, Leistungen/Angebot, Kontaktdaten (E-Mail,
      Telefon, Adresse — nur was existiert), Tonalität (Edel-Luxus,
      B2B-Pragmatisch, Tech-Spielerisch, Editorial, Brutalist, Soft-Friendly),
      Sprache (Default `de`).

   b) **Referenzen**: URLs anderer Websites (`WebFetch` — Typo, Farbe, Layout
      in 2–3 Sätzen zurückspiegeln), Screenshots/Moodboards/Logo (`Read`
      multimodal). Referenzen sind optional, aber explizit danach fragen.

   c) **`assets/` inventarisieren** (`ls -la assets/`): Logo, Fotos, Fonts,
      Grafiken. Inventar dem User zurückspiegeln („gefunden: logo.svg, 4 Fotos,
      keine Fonts"). Fehlt erwartbares Material: User auffordern, es JETZT in
      `assets/` zu legen — nicht später.

   d) **Copy-Rohmaterial**: vorhandene Texte, USPs, FAQ-Stoff, Zertifikate,
      Referenz-Kunden.

2. **Material-Gate — `AskUserQuestion`, PFLICHT:**

   - Question: *„Material komplett? Die HTML wird aus genau diesem Stand generiert."*
   - Options: *Ja, generieren* · *Nein, ich liefere noch nach*

   Bei „Nein": warten, erneut inventarisieren, Gate wiederholen. NIEMALS mit
   Teilmaterial generieren.

3. **`project.config.json` schreiben** (Repo-Root) mit dem erfassten Brand-Teil:

   ```json
   {
     "brand_name": "…",
     "brand_tagline": "…",
     "industry": "…",
     "services": ["…"],
     "contact": { "email": null, "phone": null, "address": null },
     "tonality": "…",
     "lang": "de",
     "primary_color": null,
     "style_direction": null,
     "domain": null,
     "bfsg_relevant": null,
     "stack": null,
     "deploy_target": null,
     "standards_snapshot": "<Inhalt von web-standards/.snapshot-version>"
   }
   ```

   `primary_color` + `style_direction` werden nach der Style-Wahl (Step 5)
   nachgetragen. `domain`, `bfsg_relevant`, `stack`, `deploy_target` füllt
   Setup. Nicht erfasste Kontaktfelder bleiben `null` — nie erfinden.

4. **Style-Picker generieren:** 3 distinkte Style-Direktionen — klar polar,
   nicht 3× Variationen derselben Idee. Jede mit konkreter Named Aesthetic,
   nie „modern/clean/minimalistisch".

   Direktions-Heuristik nach Branche/Tonalität (Inspiration, nicht starr):

   - B2C-Lifestyle / Wellness / Therapie → Editorial Serif · Warm Monochrome · Soft Hand-Drawn
   - B2B-Tech / SaaS → Industrial Mono · Pragmatic Clean · Tech-Playful Geometric
   - E-Commerce / Fashion → Editorial Wide-Serif · Industrial Brutalist · Soft Neutral Premium
   - Banking / Finance → Editorial Conservative · Pragmatic Trust · Modern Neutral
   - Agentur / Portfolio → Editorial Bold · Industrial Brutalist · Maximalist Color

   HTML-Mockup unter `.launchgrade/mockups/style-picker.html` — eine Datei,
   3 Sections gestapelt, je ~700–900 px, self-contained. Pro Section: Header
   `Variante A · [Named Aesthetic]` + Mini-Hero (Eyebrow, Headline mit
   Brand-Name, Sub, Primary + Secondary CTA) + Typography-Specimen (Heading +
   Body namentlich, `AaBbCc 123 — &?`) + Color-Palette (5–6 Swatches mit Hex)
   + 3 Bullets *„Was diese Variante ausmacht"*.

   Im Browser öffnen — Tool-Detection-Reihenfolge:

   1. `agent-browser open .launchgrade/mockups/style-picker.html`
   2. `mcp__claude-in-chrome__tabs_create_mcp` mit `file://`-URL
   3. Fallback: absoluten Pfad zum manuellen Öffnen ausgeben

5. **Auswahl per `AskUserQuestion`** — PFLICHT, keine nummerierte Chat-Liste:

   - Question: *„Welche Style-Direktion?"*
   - Options: *Variante A* · *Variante B* · *Variante C* · *Kombi (ich beschreibe selbst)*
   - Bei „Kombi" Nachfass: *„Was kombinieren?"* (z.B. „Typo aus B, Palette aus C")

   Danach `primary_color` (Hex + OKLCH) und `style_direction` (Named Aesthetic)
   in `project.config.json` nachtragen.

6. **Single HTML bauen — Master-Prompt ist Pflicht:**

   `.claude/skills/launchgrade-design/master-prompt.md` mit `Read` laden und
   ALLE Constraints daraus anwenden (Anti-Slop, A11y, Static-Disziplin,
   Self-Contained-Regeln, Copy-Regeln, Self-Check). Nie aus dem Gedächtnis.
   Output: `index.html` im Repo-Root.

7. **Browser-Sichtprüfung + Iteration:** `index.html` nach jedem Build im
   Browser öffnen (gleiche Tool-Detection wie Step 4). User verfeinert im
   Dialog („Hero größer", „andere Fotos") — der Skill iteriert auf der
   `index.html`. Master-Prompt-Constraints gelten in jeder Iteration.

8. **Hard Gate vor Übergabe — `AskUserQuestion`, PFLICHT:**

   Vorher: verbleibende `{{TODO: …}}`-Marker in `index.html` listen.

   - Question: *„Draft freigegeben — weiter zu /launchgrade-setup (in Form bringen)?"*
   - Options: *Ja, Setup starten* · *Nein, weiter iterieren* · *Komplett neue Direktion* (zurück zu Step 4)

   Bei „Ja" Setup nicht selbst auslösen — User triggert `/launchgrade-setup`.

Standards-Lookup: `./web-standards/AGENTS.md` im Repo (§1 HTML-Baseline, §3 A11y).

## Verhalten

- Material-Gate ist hart: ohne explizite „komplett"-Bestätigung keine Generierung.
- Master-Prompt vor JEDER Generierung lesen — Qualität ist promptversioniert,
  nicht promptabhängig.
- Bei Konflikt Brand vs. A11y: A11y gewinnt (web-standards §3).
- Material-Gate, Style-Wahl und Hard Gate: alle via `AskUserQuestion`, nie
  nummerierte Chat-Liste.
- Fakten, die nirgends im Material stehen, NIE erfinden — `{{TODO: …}}`-Marker.
- Auf Deutsch antworten, Code/Tokens/Commits auf Englisch.
- Em-dashes in user-facing Copy sparsam.

## Anti-Patterns

- ❌ Mit Teilmaterial generieren („Refs reichen schon") — Material-Gate abwarten.
- ❌ Nachgeliefertes Material still in den bestehenden Draft mischen — neue
  Iteration explizit machen (was ändert sich, was bleibt).
- ❌ DESIGN.md oder COPY.md anlegen — abgeschafft, die HTML ist die Wahrheit.
- ❌ Master-Prompt überspringen oder aus dem Gedächtnis paraphrasieren.
- ❌ „Modern, clean, minimalistisch" als Style-Beschreibung — bedeutungsleer.
- ❌ Inter als Default-Font, Lila-Gradients, Tailwind-Slate ohne Override.
- ❌ JS-Motion-Layer im Draft (IntersectionObserver, Scroll-Reveals).
- ❌ Google Fonts via `fonts.googleapis.com` einbinden.
- ❌ Daten erfinden (Telefon, Adresse, Preise, Termine).
- ❌ Stillschweigend zu Setup/Audit weitergehen — Hard Gate ist Pflicht.
- ❌ Stack-Fragen beantworten oder Scaffold anstoßen — gehört zu `launchgrade-setup`.

## Übergabe

- `index.html` im Repo-Root — self-contained, Phase 1 Static, verbleibende
  `{{TODO: …}}`-Marker explizit gelistet.
- `project.config.json` mit Brand-Teil gefüllt, Technik-Felder `null`.
- `assets/` referenziert, Style-Picker bleibt unter `.launchgrade/mockups/`
  liegen (gitignored, visuelle Ground-Truth der Wahl).
- Nächste Phase: `/launchgrade-setup` (Pflicht-Files, Stack-Wahl, CSP) —
  optional, wenn die Seite produktiv werden soll. Danach `/launchgrade-audit`.
````

- [ ] **Step 2: Verifizieren**

Run: `grep -n "DESIGN.md\|COPY.md" .claude/skills/launchgrade-design/SKILL.md`
Expected: Treffer NUR in Sätzen, die das Abschaffen beschreiben („existieren nicht mehr", „anlegen — abgeschafft", Frontmatter „No DESIGN.md/COPY.md").

Run: `grep -c "AskUserQuestion" .claude/skills/launchgrade-design/SKILL.md`
Expected: ≥ 4 (Material-Gate, Style-Wahl, Hard Gate, Verhalten).

- [ ] **Step 3: Commit**

```bash
git add .claude/skills/launchgrade-design/SKILL.md
git commit -m "feat(design-skill): rebuild as phase 1 single-HTML draft generator

Material capture with hard gate, style picker kept, generation via
versioned master prompt. DESIGN.md/COPY.md removed - index.html is
the design truth. Writes project.config.json brand part.

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

---

### Task 4: `launchgrade-setup` SKILL.md neu schreiben

**Files:**
- Rewrite: `.claude/skills/launchgrade-setup/SKILL.md`

- [ ] **Step 1: Datei komplett ersetzen mit folgendem Inhalt**

````markdown
---
name: launchgrade-setup
description: Phase 2 of the Launchgrade workflow — takes the approved single-HTML draft (index.html from launchgrade-design) and brings it into production shape. Stack choice happens here (plain HTML default, or migration to Astro/Next/SvelteKit/Nuxt with visual parity check), plus required files (robots.txt with AI-crawler defaults, sitemap.xml, llms.txt, security.txt, manifest, 404/500), complete head metas, CSP/security headers per deploy target, JSON-LD, favicons. Never changes content or look (design fidelity). Triggers on "setup", "bring into shape", "go live", "deploy", "stack", "migrate", "required files", "CSP", "robots.txt", "manifest", "security.txt", "favicon", "JSON-LD", "hreflang", "in Form bringen", "produktiv machen", "Pflicht-Files", "Setup".
---

# Launchgrade Web Setup Skill

Zweite Phase im Launchgrade-Workflow: **In Form bringen**. Nimmt den
freigegebenen Single-HTML-Draft aus `/launchgrade-design` und macht ihn
produktionsreif — Pflicht-Files, `<head>`-Metas, CSP/Headers, optional
Stack-Migration. Diese Phase ist optional: nur nötig, wenn die Seite live
gehen soll.

## Grundregel: Design-Treue

**Setup verändert weder Inhalt noch Optik.** Der Draft ist die Ground-Truth —
Setup ergänzt ausschließlich Technik (Metas, Files, Header, Struktur). Wer am
Design oder an Texten drehen will → zurück zu `/launchgrade-design`.

## Wann triggern

- Draft ist freigegeben und soll produktiv werden („in Form bringen", „live")
- Konkrete Setup-Fragen: CSP, robots.txt für AI-Bots, Manifest, Favicons,
  security.txt, hreflang, JSON-LD, Stack-Migration
- Migration eines Bestandsprojekts auf Launchgrade-Standards (technischer Teil)

Nicht triggern bei: Design-/Copy-Änderungen (→ `launchgrade-design`),
Audit/Pre-Launch-Check (→ `launchgrade-audit`), Backend-/Admin-Tasks.

## Wo die Wahrheit liegt

Standards liegen in `./web-standards/AGENTS.md` und `./web-standards/checklist.md`
im Repo-Root. Snapshot der Launchgrade Web Standards 2026, mit jedem Release
versioniert.

Relevante Kapitel für Setup:

- §1 HTML & Document Baseline
- §5 Security
- §7 Browser-Support, Responsive & i18n
- §8 PWA & Manifest
- §10 Pflicht-Dateien im Repo-Root

**Pflicht-Schritt:** Vor jeder Generierung das relevante Kapitel mit Read
laden. Nicht aus dem Gedächtnis arbeiten — Standards können sich ändern.

## Vorgehen

1. **Voraussetzungs-Check (Bash):**

   ```bash
   [ -f index.html ] && [ -f project.config.json ] && echo "DRAFT_PRESENT" || echo "DRAFT_MISSING"
   ```

   Bei `DRAFT_MISSING`: abbrechen mit Hinweis *„erst `/launchgrade-design`
   ausführen"* — kein Blind-Scaffold auf leerem Repo. Ausnahme Bestandsprojekt:
   existiert bereits eine produktive Site ohne Draft, das explizit machen und
   nur den Pflicht-File-/Header-Teil fahren.

2. **Technische Erfassung** (nur was fehlt — Brand-Daten stehen in
   `project.config.json`, nichts doppelt fragen):

   - **Domain** — Final-URL (z.B. `example.com`)
   - **hreflang** bei Mehrsprachigkeit (z.B. `de,en`)
   - **BFSG-Relevanz** — B2C-Shop / Banking / Buchung → im Output markieren,
     `accessibility-statement.md` mitgenerieren (Pflicht seit Juni 2025)
   - **Deploy-Ziel** — Vercel / Cloudflare Pages / nginx / Apache (bestimmt
     das Format der Header-Config). Default-Annahme Vercel, sonst nachfragen.

3. **Stack-Wahl per `AskUserQuestion`** — der Draft existiert bereits als
   HTML, deshalb ist Plain HTML der Default:

   | Stack | Wann nehmen |
   |---|---|
   | **Plain HTML** (Default) | Draft IST die Website. One-Pager / Landing Page ohne App-Logik. Form-Service (Formspree, Resend) reicht. Null Toolchain. |
   | **Astro** | Multi-Page, Blog/Content-Collections, beste Lighthouse-Scores, near-zero JS. |
   | **Next.js** | App-artiges: Dashboards, E-Commerce, Auth, Server Actions, SSR/ISR. |
   | **SvelteKit** | Nur bei Svelte-Präferenz im Team. |
   | **Nuxt** | Nur bei Vue-Lock-in. |

   WordPress wird nicht angeboten (kein effizienter KI-Pipeline-Workflow,
   Edge-CSP/Nonce/`llms.txt`-Tooling fehlen).

4. **Pfad A — Plain HTML (Default):**

   a) `index.html` + `assets/` nach `public/` verschieben (`git mv`) —
      `public/` ist das Publish-Dir, dort liegen die Pflicht-Files bereits.
      Relative `assets/…`-Referenzen bleiben gültig.
   b) `<head>` vervollständigen (Design-Treue: nichts Sichtbares ändern):
      canonical, Open Graph, Twitter Card, `theme-color` (= `primary_color`
      aus `project.config.json`), Favicon-Links, JSON-LD `Organization` +
      `WebSite` als `<script type="application/ld+json">`.
   c) Pflicht-File-Platzhalter via `Edit` füllen: `public/robots.txt`,
      `public/sitemap.xml`, `public/llms.txt`, `public/site.webmanifest`,
      `public/.well-known/security.txt`, `public/404.html`, `public/500.html`,
      `SECURITY.md` — Tokens: `{{BRAND_NAME}}`, `{{BRAND_TAGLINE}}`,
      `{{DOMAIN}}`, `{{LANG}}`, `{{BRAND_PRIMARY}}`, `{{CONTACT_EMAIL}}`,
      `{{SECURITY_EMAIL}}`, `{{SECURITY_EXPIRES}}` (12 Monate ab heute, ISO 8601).
   d) Security-Header-Config je Deploy-Ziel schreiben (`vercel.json` /
      `_headers` für Cloudflare Pages / nginx-Block / Apache `.htaccess`):
      CSP Level 3, HSTS, X-Content-Type-Options, Referrer-Policy,
      Permissions-Policy, `frame-ancestors 'self'` + Allowlist falls Embeds.
   e) Favicon-Set: vorhandenes `public/favicon.svg` auf Brand prüfen; PNGs
      (192/512/180) aus Logo in `assets/` ableiten oder RealFaviconGenerator
      empfehlen.

5. **Pfad B — Migration (Astro / Next / SvelteKit / Nuxt):**

   a) **Draft archivieren:** `mkdir -p .launchgrade && cp index.html .launchgrade/draft.html`
   b) **Scaffold non-interactive** (Annahmen: npm, TypeScript strict):

   | Stack | Scaffold-Befehl (non-interactive) |
   |---|---|
   | **Astro** | `npm create astro@latest . -- --template minimal --typescript strict --install --git --skip-houston --yes` |
   | **Next.js** | `npx create-next-app@latest . --typescript --tailwind --app --src-dir --import-alias "@/*" --use-npm --eslint --yes` |
   | **SvelteKit** | `npx sv create . --template minimal --types ts --no-add-ons --install npm` |
   | **Nuxt** | `npx nuxi@latest init . --packageManager npm --gitInit --force` |

   Konflikt-Handling: Scaffold läuft in nicht-leerem Dir — Meldungen zu
   existierenden Files ignorieren, Pflicht-Files werden danach sowieso
   gefüllt. Bei harten Fehlern (npm-Crash, Berechtigung): abbrechen, Fix
   vorschlagen. Nach dem Scaffold mit `ls` verifizieren, dass `package.json`
   + Framework-Config existieren. Falls kein Node 20+: Platform erkennen,
   einfachsten Installer empfehlen (`fnm`, `volta` — nicht `nvm` als Pflicht).

   c) **HTML in Components zerlegen** (Section = Component, Markup und
      Styles 1:1 übernehmen — Design-Treue):

   - **Astro:** `src/components/<Section>.astro` + `src/layouts/Base.astro` + `src/pages/index.astro`
   - **Next (App Router):** `app/components/<Section>.tsx` + `app/layout.tsx` + `app/page.tsx`
   - **SvelteKit:** `src/lib/components/<Section>.svelte` + `src/routes/+layout.svelte` + `src/routes/+page.svelte`
   - **Nuxt:** `components/<Section>.vue` + `layouts/default.vue` + `pages/index.vue`

   d) **Token-Block** aus dem Draft-`:root` nach `src/styles/tokens.css`
      (bzw. Stack-Pendant) verschieben, global einbinden. `assets/` →
      `public/assets/` verschieben, Referenzen anpassen.
   e) Root-`index.html` löschen (archiviert in `.launchgrade/draft.html`).
   f) `<head>`-Metas, Pflicht-File-Platzhalter, Header-Config, Favicons wie
      Pfad A Schritte b–e (stack-spezifisch: Next `next.config.js`-Headers,
      Astro `vercel.json` etc.).
   g) **Visueller Abgleich (Pflicht):** Dev-Server starten, Draft
      (`.launchgrade/draft.html`) und migrierte Seite im Browser vergleichen
      (Tool-Detection: `agent-browser` → Chrome MCP → Playwright → manuell).
      Abweichungen in Layout/Typo/Farbe fixen, bevor Setup „fertig" meldet.

6. **`project.config.json` ergänzen:** `domain`, `bfsg_relevant`, `stack`,
   `deploy_target` setzen. Brand-Felder unverändert lassen.

7. **Smoke-Test:**

   a) Server im Hintergrund starten:
      - Plain HTML: `npx serve public -p 8080`
      - Astro: `npm run dev` → 4321 · Next/Nuxt: `npm run dev` → 3000 · SvelteKit: `npm run dev` → 5173

      Bash mit `run_in_background: true`, 3–5 s warten.

   b) DOM-Check auf `http://localhost:<port>/`: `<title>` enthält Brand-Name,
      `<meta name="theme-color">` = `primary_color`, `<html lang>` gesetzt,
      genau 1 `<h1>`.

   c) Pflicht-Files via curl:

      ```bash
      for p in /robots.txt /sitemap.xml /llms.txt /site.webmanifest /.well-known/security.txt; do
        curl -fsSL -o /dev/null -w "%{http_code} $p\n" "http://localhost:<port>$p"
      done
      ```

      Plus inhaltlich: `robots.txt` enthält `GPTBot`, `security.txt`
      `Expires` ≥ heute, `site.webmanifest` `theme_color` === `primary_color`.

   d) Server killen — keine Zombie-Prozesse.

   Bei Fail: Skill bricht nicht ab — konkreter Fehler kommt als Aufgabe in
   den Output.

8. **Übersicht ausgeben:** Was wurde gesetzt (Pfade), gewählter Stack,
   Smoke-Test-Ergebnis, verbleibende `{{TODO: …}}`-Marker aus dem Draft,
   Hinweis: *„Vor Release `/launchgrade-audit <URL>` ausführen."*

## Verhalten

- web-standards/AGENTS.md immer lesen vor Generierung — nie aus dem Kopf.
- Design-Treue: kein sichtbares Element, keine Copy, kein Style ändern.
  Einzige Ausnahme: technisch notwendige, unsichtbare Ergänzungen (Metas,
  JSON-LD, Skip-Link falls im Draft vergessen).
- Stack-Wahl via `AskUserQuestion`, Plain HTML als Default anbieten.
- Nur Pflicht-Artefakte. Optionales (Service Worker, View Transitions,
  Speculation Rules) erst nach Nachfrage.
- Auf Deutsch antworten, Code/Header/Werte/Tokens auf Englisch.
- Bei BFSG-Relevanz aktiv darauf hinweisen (B2C-Shop / Banking / Buchung).
- Bei DSGVO-Verstößen klar markieren (Google Fonts CDN, hardcoded reCAPTCHA,
  US-Tools ohne AVV).

## Anti-Patterns

- ❌ Ohne Draft loslegen — Voraussetzungs-Check ist Pflicht, Verweis auf
  `/launchgrade-design`.
- ❌ Inhalt, Copy oder Optik des Drafts „verbessern" — Design-Treue.
- ❌ Brand-Daten erneut abfragen, die in `project.config.json` stehen.
- ❌ Stack-Migration ohne visuellen Abgleich „fertig" melden.
- ❌ Standards aus dem Kopf rezitieren statt web-standards/AGENTS.md zu lesen.
- ❌ Generische `robots.txt` ohne AI-Crawler-Konfiguration.
- ❌ CSP mit `unsafe-inline` ohne dokumentierte Begründung.
- ❌ Google Fonts via `fonts.googleapis.com` einbauen.
- ❌ Favicon nur als `.ico`.
- ❌ `security.txt` ohne `Expires` oder mit `Expires < heute`.
- ❌ `frame-ancestors 'none'` ohne Allowlist-Hinweis — bricht Whitelabel-Previews.
- ❌ Mehrere `<h1>` pro Seite.
- ❌ Bilder ohne `width`/`height`-Attribute.

## Übergabe an nächste Phase

- **`launchgrade-audit`** vor jedem Release (Lighthouse + Mozilla Observatory
  + PageSpeed Insights + Runtime-Browser-Checks).
- Design-/Copy-Änderungen jederzeit → zurück zu **`launchgrade-design`**
  (bei Migration: Änderung im Stack-Code, Draft-Archiv ist dann historisch).

## Aktualität

Die Standards haben einen Changelog am Ende der web-standards/AGENTS.md. Wenn
die Nutzerin nach einem Punkt fragt, der nach dem letzten Changelog-Eintrag
liegen könnte, kurz darauf hinweisen.
````

- [ ] **Step 2: Verifizieren**

Run: `grep -n "Erste Phase\|Foundation" .claude/skills/launchgrade-setup/SKILL.md`
Expected: keine Treffer (Setup ist jetzt „Zweite Phase … In Form bringen").

Run: `grep -c "Design-Treue" .claude/skills/launchgrade-setup/SKILL.md`
Expected: ≥ 3.

- [ ] **Step 3: Commit**

```bash
git add .claude/skills/launchgrade-setup/SKILL.md
git commit -m "feat(setup-skill): rebuild as phase 2 production-shaping skill

Requires approved single-HTML draft, stack choice moves here (plain
HTML default, migration with visual parity check), design fidelity
rule replaces foundation-no-content rule.

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

---

### Task 5: `launchgrade-audit` SKILL.md anpassen

**Files:**
- Modify: `.claude/skills/launchgrade-audit/SKILL.md`

- [ ] **Step 1: §4a grep-Pfade erweitern (3 Edits)**

Edit 1 — old_string:
```
grep -rhoE '(<script[^>]*src=|[Ss]cript[[:space:]]+src=|createElement\("script"\)[[:space:]]*[^;]*src[[:space:]]*=)[^"'\'']*["'\''][^"'\'' ]+' src/ \
```
new_string:
```
grep -rhoE '(<script[^>]*src=|[Ss]cript[[:space:]]+src=|createElement\("script"\)[[:space:]]*[^;]*src[[:space:]]*=)[^"'\'']*["'\''][^"'\'' ]+' index.html src/ public/ 2>/dev/null \
```

Edit 2 — old_string:
```
grep -rhoE '<iframe[^>]*src=["'\''][^"'\'' ]+' src/ | grep -oE 'https?://[^"'\'' ]+' | sort -u
```
new_string:
```
grep -rhoE '<iframe[^>]*src=["'\''][^"'\'' ]+' index.html src/ public/ 2>/dev/null | grep -oE 'https?://[^"'\'' ]+' | sort -u
```

Edit 3 — old_string:
```
grep -rhoE 'fetch\(["'\'']https?://[^"'\'' ]+' src/ | grep -oE 'https?://[^"'\'' ]+' | sort -u
```
new_string:
```
grep -rhoE 'fetch\(["'\'']https?://[^"'\'' ]+' index.html src/ public/ 2>/dev/null | grep -oE 'https?://[^"'\'' ]+' | sort -u
```

- [ ] **Step 2: Übergabe-Verweis aktualisieren**

old_string:
```
- Design-Qualität bleibt manueller Check via **`launchgrade-design`** (DESIGN.md vs. Output).
```
new_string:
```
- Design-Qualität bleibt manueller Check via **`launchgrade-design`** (Single-HTML-Draft als Ground-Truth vs. Live-Output).
```

- [ ] **Step 3: Verifizieren**

Run: `grep -n "DESIGN.md" .claude/skills/launchgrade-audit/SKILL.md ; grep -c "index.html src/ public/" .claude/skills/launchgrade-audit/SKILL.md`
Expected: kein `DESIGN.md`-Treffer; Zähler = 3.

- [ ] **Step 4: Commit**

```bash
git add .claude/skills/launchgrade-audit/SKILL.md
git commit -m "fix(audit-skill): cover root index.html in CSP grep, update design handoff ref

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

---

### Task 6: Root `AGENTS.md` umschreiben

**Files:**
- Rewrite: `AGENTS.md` (CLAUDE.md ist Symlink darauf — nichts Zweites anfassen)

- [ ] **Step 1: Datei komplett ersetzen mit folgendem Inhalt**

````markdown
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
````

- [ ] **Step 2: Verifizieren**

Run: `grep -n "launchgrade-setup" AGENTS.md | head -3 && grep -n "DESIGN.md" AGENTS.md && readlink CLAUDE.md`
Expected: Setup-Erwähnungen nur als Phase 2; `DESIGN.md` nur in Verbots-Kontext; `AGENTS.md` als Symlink-Ziel.

- [ ] **Step 3: Commit**

```bash
git add AGENTS.md
git commit -m "docs(agents): rewrite constitution for single-HTML-first workflow

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

---

### Task 7: `README.md` umschreiben

**Files:**
- Rewrite: `README.md`

- [ ] **Step 1: Datei komplett ersetzen mit folgendem Inhalt**

````markdown
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

That's it. No Node, no nvm, no global CLIs. The draft is a single HTML file that opens in any browser. Node only appears if you later choose a stack migration (Astro / Next.js / SvelteKit / Nuxt) — installed on demand by the agent.

## What's inside

- **Launchgrade Web Standards 2026** (`web-standards/AGENTS.md` + `checklist.md`) — versioned baseline covering Performance (Core Web Vitals 2026), Accessibility (WCAG 2.2 AA + BFSG), SEO (classic + AI-crawler era), Security (CSP, headers, cookies, auth), Privacy (GDPR / TDDDG), Motion, PWA. RFC-2119 levels (MUST / Conditional MUST / SHOULD / MAY).
- **Three Claude Code skills** (`.claude/skills/`) — Draft → Shape → Audit:
  - `launchgrade-design` — phase 1: captures material with a hard gate, 3 style directions, generates the single-HTML draft via the versioned master prompt (`master-prompt.md`). The HTML is the design truth — no DESIGN.md.
  - `launchgrade-setup` — phase 2 (optional): brings the approved draft into production shape — stack choice (plain HTML default, or migration with visual parity check), required files, CSP, head metas, JSON-LD. Never changes content or look.
  - `launchgrade-audit` — phase 3: pre-/post-launch via PageSpeed Insights + Mozilla Observatory (web-based, zero install); Lighthouse CLI optional
- **Asset drop zone** (`assets/`) — logo, photos, fonts; inventoried before generation
- **Required files** in `public/` with `{{PLACEHOLDER}}` tokens (robots.txt, sitemap.xml, llms.txt, security.txt, site.webmanifest, 404/500)
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
- Favicons as PNG/ICO — brand asset, per project
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
````

- [ ] **Step 2: Verifizieren**

Run: `grep -n "Say: \*\*\"design\"\*\*\|Say: \*\*\"setup\"\*\*" README.md && grep -n "DESIGN.md" README.md`
Expected: „Say: **"design"**" vorhanden, kein verbliebenes „Say: setup" als Einstieg; `DESIGN.md` nur in „no DESIGN.md"-Kontext.

- [ ] **Step 3: Commit**

```bash
git add README.md
git commit -m "docs(readme): describe single-HTML-first workflow, design as entry point

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

---

### Task 8: `docs/STACK_CHOICE.md` anpassen

**Files:**
- Modify: `docs/STACK_CHOICE.md`

- [ ] **Step 1: Hinweis nach der Überschrift einfügen**

old_string:
```
# Stack Choice

Which stack fits which project type? Default recommendations.
```
new_string:
```
# Stack Choice

Which stack fits which project type? Default recommendations.

> **When does this decision happen?** In phase 2 (`launchgrade-setup`), after the single-HTML draft from `/launchgrade-design` exists and is approved. Plain HTML is the default — a stack migration is opt-in, with the draft as visual ground truth.
```

- [ ] **Step 2: Generierungs-Hinweis aktualisieren**

old_string:
```
Generated by the `launchgrade-setup` skill after init.
```
new_string:
```
Filled in by the `launchgrade-setup` skill in phase 2 (after the draft is approved).
```

- [ ] **Step 3: Verifizieren + Commit**

Run: `grep -n "phase 2" docs/STACK_CHOICE.md`
Expected: 2 Treffer.

```bash
git add docs/STACK_CHOICE.md
git commit -m "docs(stack-choice): note that stack decision happens in phase 2

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

---

### Task 9: Konsistenz-Sweep über das ganze Repo

**Files:** keine neuen — reine Verifikation, ggf. Fixes.

- [ ] **Step 1: Stale-Referenzen suchen**

```bash
cd /Users/peterkasseroler/Projects/Everlast/launchgrade-web
grep -rn "DESIGN.md\|COPY.md" --exclude-dir=.git --exclude-dir=web-standards --exclude-dir=superpowers . | grep -v "docs/superpowers"
grep -rni "first.draft build\|Foundation" .claude/skills/ AGENTS.md README.md
grep -rn "launchgrade-setup" README.md AGENTS.md docs/ | grep -iv "phase 2\|setup skill\|bring\|shape\|production\|fills\|filled"
```

Expected: Erste Suche → Treffer nur in Abschaffungs-/Verbots-Sätzen („no DESIGN.md", „nicht anlegen"). Zweite Suche → keine Treffer, die Setup als erste Phase oder Design als Full-Build beschreiben. Dritte Suche → manuell prüfen, dass kein Treffer den alten „Setup zuerst"-Flow beschreibt. CONTRIBUTING.md mitprüfen: `grep -n "setup\|workflow" CONTRIBUTING.md`.

- [ ] **Step 2: Gefundene Stale-Referenzen fixen** (mit `Edit`, gleiche Stilregeln wie die Tasks oben), dann Sweep wiederholen bis sauber.

- [ ] **Step 3: Symlink-Integrität prüfen**

Run: `readlink CLAUDE.md && diff <(cat CLAUDE.md) <(cat AGENTS.md) && echo SYMLINK_OK`
Expected: `AGENTS.md` + `SYMLINK_OK`.

- [ ] **Step 4: Abschluss-Commit (nur falls Step 2 Fixes erzeugt hat)**

```bash
git add -A
git commit -m "docs: sweep remaining references to the old setup-first workflow

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

- [ ] **Step 5: Branch-Status ausgeben**

Run: `git log --oneline main..HEAD`
Expected: Spec-Commit + 7–8 Task-Commits. Danach Übergabe an den User: Review / PR / Merge gemäß superpowers:finishing-a-development-branch.

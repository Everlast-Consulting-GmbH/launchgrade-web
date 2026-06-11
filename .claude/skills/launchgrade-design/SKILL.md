---
name: launchgrade-design
description: Phase 1 of the Launchgrade workflow — design-first single-HTML draft as the visual ground truth of the site. Captures company info, references, and assets completely upfront (hard gate before generation), shows 3 style directions (user picks one), asks motion tier (Static/Micro/Cinematic with full GSAP choreography) and asset art direction (abstract graphics vs. photoreal), then generates index.html via the versioned master prompt (anti-slop, structural a11y, reduced-motion gating). Compliance hardening (self-hosting swap, metas, CSP) is deferred to launchgrade-setup. Writes project.config.json. No DESIGN.md/COPY.md — the HTML is the design truth. Triggers on "new website", "landing page", "design", "draft", "first draft", "look", "brand", "single HTML", "copy", "looks generic", "style guide", "neue Website", "Webseite erstellen", "Landing Page", "Entwurf", "erster Entwurf", "Design", "Anti-Slop", "sieht generisch aus", "Webseiten-Text".
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
     "primary_color": { "hex": null, "oklch": null },
     "style_direction": null,
     "motion_tier": null,
     "asset_style": null,
     "domain": null,
     "bfsg_relevant": null,
     "stack": null,
     "deploy_target": null,
     "standards_snapshot": "<Inhalt von web-standards/.snapshot-version>"
   }
   ```

   `primary_color.hex` + `primary_color.oklch` + `style_direction` werden nach
   der Style-Wahl (Step 5) nachgetragen, `motion_tier` + `asset_style` nach
   Step 5b/5c. `domain`, `bfsg_relevant`, `stack`,
   `deploy_target` füllt Setup. Nicht erfasste Kontaktfelder bleiben `null` —
   nie erfinden. `standards_snapshot` mit dem echten Datei-Inhalt füllen
   (`cat web-standards/.snapshot-version`), nicht den Platzhalter übernehmen.

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

   Vorher sicherstellen, dass `.launchgrade/` gitignored ist:
   `grep -q '^\.launchgrade/' .gitignore 2>/dev/null || echo '.launchgrade/' >> .gitignore`

5. **Auswahl per `AskUserQuestion`** — PFLICHT, keine nummerierte Chat-Liste:

   - Question: *„Welche Style-Direktion?"*
   - Options: *Variante A* · *Variante B* · *Variante C* · *Kombi (ich beschreibe selbst)*
   - Bei „Kombi" Nachfass: *„Was kombinieren?"* (z.B. „Typo aus B, Palette aus C")

   Danach `primary_color.hex` + `primary_color.oklch` und `style_direction`
   (Named Aesthetic) in `project.config.json` nachtragen.

   **5b · Motion-Tier per `AskUserQuestion`** — PFLICHT:

   - Question: *„Wie viel Motion soll der Draft bekommen?"*
   - Options:
     - *Static* — Layout, Typo, Farbe ohne JS-Motion (klassische Phase 1)
     - *Micro* — CSS-Transitions, Hover-/Focus-Zustände, dezente CSS-Reveals
     - *Cinematic* — volle Choreografie: GSAP + ScrollTrigger + Lenis,
       Preloader, Line-Mask-Reveals, Scrub-Parallax, Marquee, Magnetic
       Buttons, Custom Cursor, Count-Up-Stats (Rezept im Master-Prompt)
   - Enthält das Briefing Wörter wie „cinematic", „premium", „Awwwards",
     „wow": *Cinematic* als „(Empfohlen)" zuerst listen.

   **5c · Asset-Art-Direction per `AskUserQuestion`** — PFLICHT, bevor
   irgendein Asset generiert wird:

   - Question: *„Welche Bildsprache für generierte Assets?"*
   - Options:
     - *Abstrakte Grafiken* — Fine-Art-Lichtformen in Brand-Palette
       (Higgsfield `gpt_image_2`; Default bei Tier Cinematic)
     - *Fotorealistisch* — Orte/Szenen ohne Personen (Higgsfield `soul_location`)
     - *Nur eigenes Material* — ausschließlich Dateien aus `assets/`
   - Danach `motion_tier` und `asset_style` in `project.config.json`
     nachtragen. Generierungs-Rezept: Master-Prompt §Asset-Art-Direction.

6. **Single HTML bauen — Master-Prompt ist Pflicht:**

   `.claude/skills/launchgrade-design/master-prompt.md` mit `Read` laden und
   ALLE Constraints daraus anwenden (Output-Konventionen, Anti-Slop, A11y,
   Motion-Tier, Asset-Art-Direction, Copy-Regeln, Self-Check). Nie aus dem
   Gedächtnis. Output: `index.html` im Repo-Root.

7. **Browser-Sichtprüfung + Iteration:** `index.html` nach jedem Build im
   Browser öffnen — Tool-Detection-Reihenfolge: `agent-browser open index.html`
   → `mcp__claude-in-chrome__tabs_create_mcp` mit `file://`-URL → Fallback:
   absoluten Pfad ausgeben. User verfeinert im Dialog („Hero größer", „andere
   Fotos") — der Skill iteriert auf der `index.html`. Master-Prompt-Constraints
   gelten in jeder Iteration. Wurde das Projekt schon mit `/launchgrade-setup`
   in Form gebracht (Datei liegt unter `public/index.html`, Root-`index.html`
   existiert nicht), dort iterieren — KEINE zweite Root-Datei anlegen.

8. **Hard Gate vor Übergabe — `AskUserQuestion`, PFLICHT:**

   Vorher: verbleibende `{{TODO: …}}`-Marker in `index.html` listen.

   - Question: *„Draft freigegeben — weiter zu /launchgrade-setup (in Form bringen)?"*
   - Options: *Ja, Setup starten* · *Nein, weiter iterieren* · *Komplett neue Direktion* (zurück zu Step 4)

   Bei „Komplett neue Direktion": vorher in `project.config.json`
   `style_direction` auf `null` und `primary_color` auf
   `{ "hex": null, "oklch": null }` zurücksetzen — die alte Wahl gilt nicht
   weiter. Bei „Ja" Setup nicht selbst auslösen — User triggert `/launchgrade-setup`.

Standards-Lookup: `./web-standards/AGENTS.md` im Repo (§1 HTML-Baseline, §3 A11y).

## Verhalten

- Material-Gate ist hart: ohne explizite „komplett"-Bestätigung keine Generierung.
- Master-Prompt vor JEDER Generierung lesen — Qualität ist promptversioniert,
  nicht promptabhängig.
- **Design-first:** Die Design-Phase ist kreativ, nicht compliance-getrieben.
  Self-Hosting-Swap, Metas, CSP und Pflicht-Files macht `/launchgrade-setup`
  als Retrofit ohne Pixel-Änderung. Im Draft sind Fontshare-/jsdelivr-CDNs
  erlaubt (nie `fonts.googleapis.com`) — als Setup-TODO im Header notieren.
- Struktur-A11y ist auch im Draft nicht verhandelbar (lang, ein h1, Landmarks,
  alt, focus-visible, reduced-motion-Fallback) — sie kostet kein Design.
  Kontrast ist im Draft Hinweis, die harte Prüfung macht `/launchgrade-audit`.
- Material-Gate, Style-Wahl, Motion-Tier, Art-Direction und Hard Gate: alle
  via `AskUserQuestion`, nie nummerierte Chat-Liste.
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
- ❌ Cinematic-Briefing („Awwwards", „premium", „wow") mit Static-Zurückhaltung
  beantworten — gewähltes Motion-Tier voll ausspielen.
- ❌ Motion ohne `prefers-reduced-motion`-Bail-out oder mit Inhalten, die ohne
  JS unsichtbar bleiben.
- ❌ Google Fonts via `fonts.googleapis.com` einbinden.
- ❌ Daten erfinden (Telefon, Adresse, Preise, Termine).
- ❌ Stillschweigend zu Setup/Audit weitergehen — Hard Gate ist Pflicht.
- ❌ Stack-Fragen beantworten oder Scaffold anstoßen — gehört zu `launchgrade-setup`.

## Übergabe

- `index.html` im Repo-Root — im gewählten Motion-Tier, verbleibende
  `{{TODO: …}}`-Marker und Setup-TODOs (CDN-Swaps) explizit gelistet.
- `project.config.json` mit Brand-Teil gefüllt, Technik-Felder `null`.
- `assets/` referenziert, Style-Picker bleibt unter `.launchgrade/mockups/`
  liegen (gitignored, visuelle Ground-Truth der Wahl).
- Motion-Umfang folgt dem gewählten Tier (Static / Micro / Cinematic);
  `prefers-reduced-motion`-Fallback ist in jedem Tier Pflicht
  (web-standards §9).
- Nächste Phase: `/launchgrade-setup` (Pflicht-Files, Stack-Wahl, CSP) —
  optional, wenn die Seite produktiv werden soll. Danach `/launchgrade-audit`.

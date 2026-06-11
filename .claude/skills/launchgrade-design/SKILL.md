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

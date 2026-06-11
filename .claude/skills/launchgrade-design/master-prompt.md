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

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
3. dem gewählten Motion-Tier und der Asset-Art-Direction (SKILL.md Step 5b/5c)
4. dem Inventar des `assets/`-Ordners
5. dem erfassten Copy-Rohmaterial

Alle fünf Inputs müssen vorliegen. Fehlt einer: zurück zur Material-Erfassung,
nicht improvisieren.

**Grundsatz Design-first:** Diese Phase optimiert auf visuelle Klasse, nicht
auf Compliance. Technik- und Rechts-Hardening (Self-Hosting-Swap, Metas,
JSON-LD, CSP, Pflicht-Files) macht `/launchgrade-setup` danach als Retrofit
ohne Pixel-Änderung. Was hier trotzdem Pflicht bleibt, ist Struktur-A11y —
weil sie beim Schreiben des HTML nichts kostet und beim Nachrüsten teuer ist.

## Output-Konventionen

- Eine Datei: `index.html`, ohne Build-Step im Browser lauffähig
- Kommentar-Header am Dateianfang:

  ```html
  <!--
    Brand-DNA: <3-5 Adjektive die die Brand IST> | NICHT: <3-5 Adjektive>
    Named Aesthetic: <konkret, z.B. "Editorial Serif nach Mr-Porter-Vorbild">
    Style-Direktion: <Variante A/B/C aus dem Style-Picker>
    Motion-Tier: <Static | Micro | Cinematic> · Asset-Style: <Grafik | Foto | eigenes Material>
    Setup-TODOs: <CDN-Referenzen, die Setup self-hosten soll — oder "keine">
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

- `<head>`-Minimum: charset, viewport (`width=device-width, initial-scale=1` —
  NIE `user-scalable=no` oder `maximum-scale`, bricht WCAG 1.4.4), `<title>`,
  `<meta name="description">`, `<meta name="color-scheme">`. Bewusste Lücke:
  canonical, OG, JSON-LD und `theme-color` ergänzt später Setup — im Draft
  nicht vorgreifen.
- Bilder relativ aus `assets/` (`<img src="assets/...">`), immer mit
  `width`/`height`-Attributen und sinnvollem `alt`. Nichts base64-inlinen.
- Responsive ohne horizontalen Overflow ab 320 px Viewport-Breite —
  `min-width: 0` auf Flex-/Grid-Kindern wo nötig; das Layout bricht auf
  Mobile-Breiten nicht.
- Fonts: self-hosted via `@font-face` aus `assets/` ODER System-Stack ODER
  (nur im Draft) Fontshare-CDN — dann als Setup-TODO im Kommentar-Header.
  NIE `fonts.googleapis.com`.
- Motion-Libraries (nur Tier Cinematic): GSAP + ScrollTrigger + Lenis,
  bevorzugt self-hosted in `assets/vendor/`, im Draft auch jsdelivr-CDN
  erlaubt (Setup-TODO notieren). Keine weiteren Dependencies.

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
- Landmarks: `<header>`, `<nav aria-label="Hauptnavigation">`, `<main id="main">`, `<footer>`
- Jede `<section>` hat eine Überschrift (`<h2>`/`<h3>`) oder `aria-labelledby`
- Anchor-Targets bekommen `scroll-margin-top`, falls ein Sticky-Header
  existiert — fokussierter Inhalt darf nicht verdeckt werden (WCAG 2.4.11)
- Genau EIN `<h1>`, logische Heading-Hierarchie ohne Sprünge
- Kontrast: Fließtext ≥ 4.5:1, Large Text / UI ≥ 3:1 anstreben — Token-Paare
  entsprechend wählen; bewusste Brand-Abweichungen im Kommentar-Header
  notieren, die harte Prüfung macht `/launchgrade-audit`
- `:focus-visible`-Indicator für alle interaktiven Elemente
- Target-Size ≥ 24×24 px
- `prefers-reduced-motion: reduce` defensiv respektieren — Transitions und
  Animations darin deaktivieren (auch wenn der Draft kaum Motion hat)

## Motion-Tiers (Wahl aus SKILL.md Step 5b — im Kommentar-Header dokumentieren)

**Tier Static** — kein JS-Motion-Layer (kein IntersectionObserver, keine
Scroll-Reveals, keine Tween-Animations). CSS-only `:hover`/`:focus-visible`
erlaubt. JS nur wenn funktional zwingend (Mobile-Nav-Toggle), minimal, inline.

**Tier Micro** — zusätzlich CSS-Transitions für Hover-/Focus-/Detail-Zustände
und dezente CSS-Reveals. Weiterhin kein JS-Scroll-Layer.

**Tier Cinematic** — volle Choreografie. Pflicht-Repertoire (nicht optional —
ein Cinematic-Briefing mit halber Kraft zu beantworten ist der Fehlermodus):

- Lenis Smooth-Scroll + GSAP ScrollTrigger; Anker-Navigation über Lenis
- Preloader: Wortmarke mit Letter-Stagger + Prozent-Counter, übergehend in
  eine Hero-Intro-Timeline (Media-Scale-In, Line-Masks, Fades)
- Line-Mask-Reveals (`.line`/`.line__inner`, translateY 112% → 0) für Hero-
  und CTA-Headlines
- Scroll-Reveals mit Blur-Resolve (y + opacity + filter), `once: true`
- Scrub-Parallax: Hero-Media, Bilder in Karten, Giant-Footer-Word
- Marquee (Track mit 2× Inhalt, xPercent -50, linearer Loop)
- Magnetic Buttons + Custom Cursor — NUR hinter `(pointer: fine) and (hover: hover)`
- Count-Up-Stats, Glow-Drift, Grain-Overlay, Island-Nav, Bezel-Cards
  (Double-Frame: Hairline-Shell + Innenradius), Fullscreen-Menü mit Stagger

**Hard Rules für JEDES Tier:**

- `prefers-reduced-motion: reduce` ⇒ Choreografie komplett aus (Early-Bail im
  JS), Inhalt sofort sichtbar, Preloader wird entfernt oder nie angezeigt
- Ohne JS bleibt ALLES sichtbar und bedienbar — Initial-Hidden-States nie im
  CSS-Default, nur via `gsap.set` bzw. hinter einer per JS gesetzten
  `html.motion`-Klasse
- Nur `transform` / `opacity` / `filter` animieren — kein Layout-Thrashing

## Asset-Art-Direction (Wahl aus SKILL.md Step 5c)

- **Abstrakte Grafiken** (Default bei Tier Cinematic): Higgsfield `gpt_image_2`,
  `--resolution 2k`, alle Jobs parallel. Gemeinsamer STYLE-Suffix pro Set, z.B.:
  *„abstract graphic fine-art, NOT photorealistic, pure black background,
  volumetric light, cinematic chiaroscuro, subtle film grain, <Brand-Farben
  konkret benennen>, no text, no watermark, no letters"*. Pro Sektion ein
  Motiv, das die Brand-Story abstrakt erzählt (Pfad, Loop, Schwelle, Aufstieg) —
  keine Deko-Abstraktion ohne Bedeutung.
- **Fotorealistisch**: Higgsfield `soul_location` für Orte/Szenen ohne
  Personen. Jedes Ergebnis visuell prüfen; bei ungewollten Personen oder
  Artefakten neu generieren, nie croppen-und-hoffen.
- Nachbearbeitung: PNG → JPEG (`sips`, Quality ~80), echte Pixelmaße in die
  `width`/`height`-Attribute übernehmen.

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
als Platzhalter-Anker `#impressum` / `#datenschutz` — DE-Pflichtangaben,
§ 5 DDG / DSGVO Art. 13, nicht optional). Keine Sektion erfinden,
für die kein Material existiert.

## Self-Check vor Abgabe (alle Punkte prüfen, bei Fail nachbessern)

1. Datei öffnet ohne Build-Step im Browser, keine 404 auf `assets/`-Referenzen,
   kein horizontaler Overflow bei 320–1920 px (Runtime-Check im Browser)
2. `<html lang>` gesetzt; genau ein `<h1>`; Landmarks vollständig; Skip-Link vorhanden
3. Kein `fonts.googleapis.com`; andere CDN-Referenzen (Fontshare, jsdelivr)
   nur im Draft erlaubt und als Setup-TODO im Kommentar-Header gelistet
3b. Bei Motion (Tier Micro/Cinematic): reduced-motion-Bail-out vorhanden UND
   ohne JS ist jeder Inhalt sichtbar (Check: `html.motion`-Klasse entfernen)
4. Alle Bilder mit `width`/`height`/`alt`
5. Token-Block vollständig — keine hardcoded Farben außerhalb von `:root`
   (gilt auch für Inline-`style=`-Attribute)
6. Kein erfundener Fakt — alle Lücken als `{{TODO: …}}` markiert
7. Kommentar-Header vollständig

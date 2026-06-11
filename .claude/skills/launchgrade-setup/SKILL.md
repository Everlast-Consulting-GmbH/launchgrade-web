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

   - Question: *„Wie soll die Seite produktiv werden?"*
   - Options: *Plain HTML (Default)* · *Astro* · *Next.js* · *SvelteKit / Nuxt*

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

   a) Draft ins Publish-Dir verschieben — exakt:
      `git mv index.html public/index.html && git mv assets public/assets`.
      `public/` ist das Publish-Dir, dort liegen die Pflicht-Files bereits.
      Relative `assets/…`-Referenzen bleiben gültig, weil beide zusammen
      umziehen.
   b) `<head>` vervollständigen (Design-Treue: nichts Sichtbares ändern):
      canonical, Open Graph, Twitter Card, `theme-color` (= `primary_color.hex`
      aus `project.config.json`), Favicon-Links, JSON-LD `Organization` +
      `WebSite` als `<script type="application/ld+json">`.
   c) Pflicht-File-Platzhalter via `Edit` füllen: `public/robots.txt`,
      `public/sitemap.xml`, `public/llms.txt`, `public/site.webmanifest`,
      `public/.well-known/security.txt`, `public/404.html`, `public/500.html`,
      `SECURITY.md` — Tokens: `{{BRAND_NAME}}`, `{{BRAND_TAGLINE}}`,
      `{{DOMAIN}}`, `{{LANG}}`, `{{BRAND_PRIMARY}}` (= `primary_color.hex`
      aus `project.config.json`), `{{CONTACT_EMAIL}}`, `{{SECURITY_EMAIL}}`,
      `{{SECURITY_EXPIRES}}` (12 Monate ab heute, ISO 8601). Übrige Werte
      kommen aus `project.config.json` + Step 2 — nichts erfinden.
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
   | **Nuxt** | `npx nuxi@latest init . --packageManager npm --no-gitInit --force` |

   CLI-Flags ändern sich mit Releases — bei Fehlern zuerst
   `npm create astro@latest -- --help` (bzw. Stack-Pendant) prüfen und den
   Befehl anpassen. Konflikt-Handling: Scaffold läuft in nicht-leerem Dir —
   Meldungen zu existierenden Files ignorieren, Pflicht-Files werden danach
   sowieso gefüllt. Bei harten Fehlern (npm-Crash, Berechtigung): abbrechen, Fix
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
      verschieben (Nuxt: `assets/styles/tokens.css`), global einbinden.
      `assets/` → `public/assets/` verschieben, Referenzen anpassen.
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

   a) Vorab Port prüfen: `portmon --json` (falls installiert, sonst
      `lsof -i :<port>`) — ist der Zielport belegt, anderen Port wählen oder
      den belegenden Prozess nur nach User-Rückfrage beenden. Dann Server im
      Hintergrund starten:
      - Plain HTML: `npx serve public -p 8080`
      - Astro: `npm run dev` → 4321 · Next/Nuxt: `npm run dev` → 3000 · SvelteKit: `npm run dev` → 5173

      Bash mit `run_in_background: true`, 3–5 s warten. `<port>` in den
      folgenden Checks = der hier tatsächlich verwendete Port.

   b) DOM-Check auf `http://localhost:<port>/` — für statisches HTML per curl:

      ```bash
      HTML=$(curl -s "http://localhost:<port>/")
      echo "$HTML" | grep -c "<h1"                      # erwartet: 1
      echo "$HTML" | grep -o '<html[^>]*lang="[^"]*"'   # lang gesetzt
      echo "$HTML" | grep -o '<title>[^<]*</title>'     # enthält Brand-Name
      echo "$HTML" | grep -o 'name="theme-color"[^>]*'  # = primary_color.hex
      ```

      Bei Framework-Builds (Next/Nuxt, client-gerenderte Anteile) statt curl
      den realen Browser nehmen — Tool-Detection: `agent-browser` → Chrome MCP
      → Playwright → manuell.

   c) Pflicht-Files via curl:

      ```bash
      for p in /robots.txt /sitemap.xml /llms.txt /site.webmanifest /.well-known/security.txt; do
        curl -fsSL -o /dev/null -w "%{http_code} $p\n" "http://localhost:<port>$p"
      done
      ```

      Plus inhaltlich: `robots.txt` enthält `GPTBot`, `security.txt`
      `Expires` ≥ heute, `site.webmanifest` `theme_color` === `primary_color.hex`.

   d) Server killen — gezielt per PID des Hintergrundprozesses oder
      `kill $(lsof -ti :<port>)`. Kein pauschales `pkill node`.

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

# Launchgrade Pre-Launch-Checkliste

**Version 1.5 · Mai 2026**

Kompakte Version der Pflicht-Anforderungen aus [AGENTS.md](./AGENTS.md). Jeder offene MUST-Punkt ist ein Launch-Blocker. SHOULD/Conditional MUST/MAY sind entsprechend markiert.

**Projekt:** ___________________ · **Datum:** ___________________ · **Auditor:** ___________________

---

## Performance

- [ ] **LCP < 2.5 s** (Mobile, CrUX field data oder PSI)
- [ ] **INP < 200 ms**
- [ ] **CLS < 0.1**
- [ ] Lighthouse Accessibility ≥ 95
- [ ] Lighthouse Best Practices ≥ 95
- [ ] Lighthouse SEO ≥ 95
- [ ] Lighthouse Performance ≥ 90 Mobile / ≥ 95 Desktop _(SHOULD — bei Pflicht-3rd-Party dokumentieren)_
- [ ] HTTP/2+ oder HTTP/3, TLS 1.2+
- [ ] Brotli für Text-Assets aktiv
- [ ] CDN- oder Edge-Hosting _(SHOULD; Conditional MUST bei intl. Zielgruppe / hohem Traffic)_
- [ ] Responsive Images mit `srcset`/`sizes` und komprimierten Formaten _(AVIF primary + WebP fallback via `<picture>` SHOULD)_
- [ ] `width` und `height` auf jedem `<img>`
- [ ] `loading="lazy"` below the fold, `fetchpriority="high"` für LCP-Image
- [ ] Fonts: nur woff2, self-hosted, `font-display: swap`, max. 2 preload
- [ ] Caching: hashed assets `Cache-Control: public, max-age=31536000, immutable`
- [ ] HTML kurze TTL oder `no-cache` + `stale-while-revalidate`
- [ ] Initial JS Mobile < 170 KB compressed _(SHOULD)_
- [ ] Max. 4 Third-Party-Scripts _(SHOULD)_
- [ ] Speculation Rules + View Transitions evaluiert _(MAY — Optimierung, kein Standard)_

---

## Accessibility (WCAG 2.2 AA)

- [ ] axe DevTools: 0 Violations
- [ ] Color Contrast: 4.5:1 Body-Text, 3:1 Large Text / UI / Focus-Indikator
- [ ] Genau ein sichtbarer `<h1>` pro Seite _(SHOULD / Launchgrade-Default)_
- [ ] Headings bilden nachvollziehbare Inhaltsstruktur (keine Sprünge) _(SHOULD)_
- [ ] Landmarks vorhanden: `<header>`, `<nav>`, `<main>`, `<footer>`
- [ ] Skip-Link ist erstes fokussierbares Element
- [ ] Alle interaktiven Elemente per Tab erreichbar
- [ ] Sichtbarer `:focus-visible`-State (kein `outline: none` ohne Ersatz)
- [ ] Tab-Order = visuelle Reihenfolge
- [ ] Target Size ≥ 24×24 CSS-Pixel (außer Inline-Text-Links)
- [ ] Forms: `<label for>`, `autocomplete`, Fehler mit `aria-describedby`
- [ ] Alt-Text auf allen `<img>` (leer `alt=""` für dekorativ)
- [ ] Video: Captions, Pause-Control bei Autoplay > 5 s
- [ ] `prefers-reduced-motion` respektiert
- [ ] `<html lang="de">` korrekt gesetzt
- [ ] Falls B2C im EAA-/BFSG-Geltungsbereich (und kein Kleinstunternehmer): BFSG-Konformitätserklärung mit Anlage-3-Pflichtinhalten (Standard, Methode, Datum, bekannte Barrieren, Alternativen) + Feedback-Mechanismus auf Site

---

## Motion & Animation

- [ ] Duration-Tokens als CSS Custom Properties definiert: `--motion-duration-instant/short/base/long` (100/200/300/500 ms) — §9.1
- [ ] Easing-Tokens als CSS Custom Properties definiert: `--motion-ease-standard/out/in/emphasized` — §9.2
- [ ] Animationen nutzen ausschließlich `transform`, `opacity`, `filter` — keine Layout-Properties (`width`, `height`, `top`, `left`, `margin`) — §9.3
- [ ] `will-change` nicht statisch im CSS auf >5 Elementen (nur via JS bei Trigger, danach entfernen) — §9.4
- [ ] `@media (prefers-reduced-motion: reduce)` Block vorhanden, ersetzt Bewegung durch Opacity ≤150 ms — §9.5
- [ ] Auto-Animationen >5 s pausierbar (WCAG 2.2.2 A); Parallax / Auto-Rotate bei `reduced-motion` deaktiviert
- [ ] Scroll-driven animations (`animation-timeline`) nur hinter `@supports` als Progressive Enhancement _(SHOULD)_ — §9.6
- [ ] View Transitions: same-document genutzt wo sinnvoll _(SHOULD)_ — §9.7
- [ ] Animations-Framework gewählt nach Hierarchie §9.8 (CSS-native first, dann Motion / GSAP / Anime.js / AutoAnimate); kein VelocityJS, kein jQuery-Animate
- [ ] DevTools Performance: 60 fps unter Mobile-Throttle bei aktiven Animationen — §9.4

---

## SEO

- [ ] `<title>` eindeutig pro Seite (~50–60 Zeichen, Pixel-Breite zählt — SHOULD)
- [ ] `<meta name="description">` eindeutig pro Seite (~120–160 Zeichen — SHOULD)
- [ ] Jede indexierbare Seite hat ein Primary Keyword und eine klare Suchintention
- [ ] Wichtige Seiten sind in `seo/strategy.md` gemappt
- [ ] Keine zwei Seiten bedienen dieselbe Suchintention ohne klare Abgrenzung
- [ ] Kommerzielle Seiten haben passenden CTA und interne Links zu relevanten Cluster-/Pillar-Seiten
- [ ] Blog-/Ratgeberseiten wurden gegen `seo/brief-template.md` geplant oder geprüft
- [ ] `<link rel="canonical">` pro Seite, selbst-referenzierend, absolute URL
- [ ] Open Graph komplett: og:title, og:description, og:url, og:type, og:locale, og:image (1200×630), og:image:width/height/alt
- [ ] X Card (vormals Twitter Card): `twitter:card="summary_large_image"` + Title/Description/Image
- [ ] hreflang symmetrisch + x-default (bei > 1 Sprache)
- [ ] JSON-LD: Organization, WebSite, BreadcrumbList _(SearchAction nur bei echter Site-Suche)_
- [ ] Rich Results Test grün
- [ ] `/robots.txt` mit `Sitemap:`-Direktive
- [ ] `/sitemap.xml` nur indexierbare URLs, valide `lastmod`
- [ ] AI-Crawler-Policy bewusst gesetzt (Default: AI-Search-Bots erlauben)
- [ ] HTTPS-only, HTTP → 301 → HTTPS
- [ ] Trailing-Slash konsistent
- [ ] 404 = echter 404-Status, nicht Soft-404
- [ ] noindex auf Filter / interne Suche / Thank-you-Pages

---

## Security

- [ ] SSL Labs ≥ A
- [ ] securityheaders.com ≥ A
- [ ] Mozilla Observatory ≥ A
- [ ] HSTS aktiv: bei stabilem HTTPS-Betrieb Ziel `max-age=63072000; includeSubDomains; preload`. Rollout-Plan: erst `max-age=2592000`, nach 4 Wochen Stabil-Betrieb auf 2 Jahre + Preload-Submit. Bei Erstaudit: mindestens `max-age=2592000` Pflicht.
- [ ] CSP gesetzt — Marketing-Baseline mit `script-src 'self'` + `script-src-attr 'none'` (MUST) / strict Nonce-CSP bei Login/Shop/Payment (MUST)
- [ ] CSP Reporting aktiv und überwacht
- [ ] `X-Content-Type-Options: nosniff`
- [ ] `Referrer-Policy: strict-origin-when-cross-origin`
- [ ] `Permissions-Policy` restriktiv gesetzt
- [ ] `Cross-Origin-Opener-Policy: same-origin`
- [ ] `Cross-Origin-Embedder-Policy: require-corp` _(MAY — nur bei Cross-Origin-Isolation-Bedarf, bricht sonst Embeds)_
- [ ] Cookies: `Secure`, `HttpOnly`, `SameSite=Lax`/`Strict`, `__Host-` für Auth
- [ ] SRI auf CDN-Skripten _(SHOULD — bevorzugt: selbst hosten)_
- [ ] Keine Mixed Content Warnings
- [ ] CSRF-Token bei state-changing Requests
- [ ] Rate-Limiting auf Forms + Login-Endpoints
- [ ] DSGVO-konformer Captcha (Turnstile / Friendly / hCaptcha), kein reCAPTCHA ohne Consent
- [ ] Server-seitige Input-Validierung (Zod / TypeBox / Valibot)
- [ ] Dependabot / Renovate aktiv
- [ ] Keine offenen Critical CVEs (`npm audit --audit-level=critical`)
- [ ] `/.well-known/security.txt` mit gültigem `Expires` (≤ 1 Jahr empfohlen)
- [ ] CAA-DNS-Record gesetzt _(SHOULD)_
- [ ] DNSSEC aktiviert _(SHOULD)_
- [ ] Bei Login-Sites: Passkeys/WebAuthn angeboten _(SHOULD)_
- [ ] Bei Login-Sites: `/.well-known/change-password` redirected korrekt _(SHOULD)_

---

## Privacy / DSGVO (DE)

- [ ] Cookie-Banner: Opt-in, "Alles ablehnen" gleichrangig, granular, kein Pre-Checked
- [ ] Consent-Widerruf jederzeit möglich (Button im Footer)
- [ ] Consent-Logging aktiv
- [ ] Vor Consent keine nicht-notwendige Speicherung / Auslesung / personenbezogene Tracking-Verarbeitung (Inkognito-Netzwerk-Tab geprüft)
- [ ] Bei Google Ads / GA4 / Floodlight: **Google Consent Mode v2** korrekt verdrahtet (4 Signale) _(Conditional MUST)_
- [ ] Bei Einsatz von Consent Mode Advanced: rechtliche Prüfung dokumentiert
- [ ] Falls Programmatic Ads / AdSense / Ad Manager: TCF v2.3-konforme CMP
- [ ] Google Fonts lokal gehostet
- [ ] Embeds (YouTube/Vimeo/Maps) per Two-Click oder Privacy-Mode + Consent
- [ ] Analytics: cookieless / first-party bevorzugt oder consent-gated
- [ ] Impressum vollständig: ladungsfähige Adresse, E-Mail + **Telefon oder gleichwertig schneller zweiter Kanal** (EuGH C-298/07: unmittelbare + effiziente Kommunikation), Register + Nummer + Gericht, USt-ID (§ 27a UStG), bei reglementierten Berufen Kammer + Berufsbezeichnung + Verleihungsstaat
- [ ] Datenschutzerklärung: alle Tools, Rechtsgrundlagen, Speicherdauern, Empfänger, Drittlandtransfer, AVV-Hinweise
- [ ] Keine PII in URLs
- [ ] Keine PII in Server-Logs / Sentry-Bodies
- [ ] IP-Anonymisierung in Analytics
- [ ] Log-Retention ≤ 90 Tage

---

## Files & Misc

- [ ] `<!DOCTYPE html>`, `<html lang="de">`, `<meta charset="UTF-8">` als erstes Meta
- [ ] Viewport-Meta ohne `user-scalable=no`
- [ ] `color-scheme` + `theme-color` (light + dark)
- [ ] Favicon: `favicon.ico`, `favicon.svg`, `apple-touch-icon.png` (180×180) _(PWA-Icons icon-192/512/maskable SHOULD)_
- [ ] `site.webmanifest` valide _(SHOULD bei klassischen Sites, MUST bei installierbaren PWAs)_
- [ ] `/404.html` und `/500.html` statisch und funktional
- [ ] AGENTS.md im Projekt-Repo (Setup, Build, Test, Conventions)

---

## Abnahme

**Audit-Ergebnis:** ☐ Bestanden ☐ Bestanden mit Auflagen ☐ Nicht bestanden

**Offene Auflagen / Begründungen:**

```
...
```

**Freigabe Launch:** ___________________ (Datum, Unterschrift)

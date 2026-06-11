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

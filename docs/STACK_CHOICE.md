# Stack Choice

Which stack fits which project type? Default recommendations.

> **When does this decision happen?** In phase 2 (`launchgrade-setup`), after the single-HTML draft from `/launchgrade-design` exists and is approved. Plain HTML is the default — a stack migration is opt-in, with the draft as visual ground truth.

## Decision matrix

If you migrate beyond plain HTML, use this matrix to pick the target stack:

| Project type                        | Migration default    | Alternatives                      | Rationale                                                |
| ---------------------------------- | -------------------- | --------------------------------- | -------------------------------------------------------- |
| **Marketing site / landing page**  | **Astro**            | Next.js, plain HTML               | Fast, static, good SEO, minimal JS footprint             |
| **Blog / magazine**                | **Astro**            | Hugo, WordPress, Ghost            | Content collections, MDX, build-time RSS                 |
| **Online shop (small/medium)**     | **Shopify**          | Medusa (custom), WooCommerce      | Out-of-the-box BFSG path easier, low maintenance load    |
| **Online shop (custom, headless)** | **Next.js + Medusa** | Astro + Snipcart, Remix + Shopify | When logic is custom or brand-heavy                      |
| **Booking system**                 | **Next.js**          | Nuxt, SvelteKit                   | App character, lots of state, auth, BFSG-relevant        |
| **SaaS marketing frontend**        | **Next.js**          | Astro with React islands          | Branding + conversion + occasional demo login            |
| **Multilingual corporate site**    | **Astro with i18n**  | Next.js App Router, SvelteKit     | Static + hreflang + low latency                          |
| **Documentation**                  | **Astro Starlight**  | Mintlify, Docusaurus              | OSS standard 2026, full domain control                   |

## Rules of thumb

- **Static where possible** — Astro, plain HTML, or Next.js with `output: 'export'`
- **App only where required** — auth, dynamic data, personalization
- **WordPress only when**: the client insists or an editorial team already uses it
- **For BFSG relevance** (shops, banking, booking): add `@axe-core/cli` in CI, manual screen-reader pass, BITV test for larger projects

## What stays the same (stack-agnostic)

Regardless of stack, these required artifacts always exist:

- `/robots.txt` with AI-crawler defaults
- `/sitemap.xml`
- `/llms.txt`
- `/.well-known/security.txt`
- `/site.webmanifest`
- Favicon set
- `/404.html`, `/500.html`
- CSP Level 3 with nonce
- HSTS, X-Frame-Options, Referrer-Policy, Permissions-Policy
- JSON-LD Organization + WebSite

Filled in by the `launchgrade-setup` skill in phase 2 (after the draft is approved).

## Hosting defaults

| Stack     | Default hosting                              |
| --------- | -------------------------------------------- |
| Astro     | Cloudflare Pages, Vercel, Netlify            |
| Next.js   | Vercel (or self-hosted, Hetzner + Coolify)   |
| WordPress | Cloudways, Raidboxes                         |
| Shopify   | Shopify (hosted)                             |

Vercel default: ships TLS, HTTP/3, edge-cached headers — Observatory A+ achievable with minimal config.

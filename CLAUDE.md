# remtheory.com — CLAUDE.md

## What this is
Rachel Murray's personal portfolio site. Single-page, static HTML/CSS — no build step, no framework.

## Live URL
https://remtheory.com

## Hosting & deployment
- **Platform**: Cloudflare Pages
- **Account**: remtheory@gmail.com
- **Method**: Currently direct-upload (bypassing GitHub). TODO: Connect a GitHub repo for proper version control and automatic deploys.
- **Security headers**: Handled by `_headers` file (Cloudflare Pages picks this up automatically on deploy)

## Tech stack
- Plain HTML (`index.html`) + CSS (`style.css`)
- Google Fonts: Fraunces (display) + Plus Jakarta Sans (body)
- Contact form: Formspree endpoint `xlgpwapv`
- No JavaScript framework, no build process

## Key files
| File | Purpose |
|---|---|
| `index.html` | Entire site (single page) |
| `style.css` | All styles |
| `_headers` | Cloudflare Pages security headers |
| `robots.txt` | Crawler instructions |
| `sitemap.xml` | Sitemap for search engines |
| `SITE_CHECKLIST.md` | Security & SEO reference for all sites |
| `rachel-murray-2026.jpeg` | Hero photo |
| `rachel-murray-2023.jpg` | About section photo |

## External services
- **Formspree** (contact form submissions) — no API key in code, endpoint is public by design
- **Google Fonts** — no API key needed

## How to update
1. Edit `index.html` or `style.css`
2. Deploy: drag the folder into Cloudflare Pages dashboard, or push to GitHub once connected

## SEO & security status
All items from SITE_CHECKLIST.md are complete. See the checklist for ongoing maintenance items.

## TODO
- [ ] Create GitHub repo (`remtheory`) and connect to Cloudflare Pages for auto-deploy
- [ ] Register with Google Search Console
- [ ] Add a privacy policy page (Formspree stores submitted emails)
- [ ] Convert photos to WebP for better performance

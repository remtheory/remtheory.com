# Site Security & SEO Checklist
_Rachel Murray / remtheory — reference for all sites and future builds_

---

## Quick-start: files every static site needs

| File | Purpose |
|---|---|
| `_headers` | Security headers (Cloudflare Pages / Netlify) |
| `vercel.json` | Security headers (Vercel) |
| `robots.txt` | Tell crawlers what to index |
| `sitemap.xml` | Help crawlers find all your pages |

---

## 1. SEO

### `<head>` must-haves
- [ ] `<title>` — unique per page, ~60 chars
- [ ] `<meta name="description">` — 140–160 chars, plain language
- [ ] `<meta name="robots" content="index, follow">` — explicit is better than assumed
- [ ] `<link rel="canonical" href="https://yourdomain.com/">` — prevents duplicate-content penalties

### Social sharing (Open Graph + Twitter Card)
- [ ] `og:type`, `og:url`, `og:title`, `og:description`, `og:image`
- [ ] `twitter:card`, `twitter:title`, `twitter:description`, `twitter:image`
- [ ] OG image should be at least 1200×630px and an absolute URL

### Structured data (JSON-LD)
- [ ] `Person` schema for personal/portfolio sites
- [ ] `Article` schema for blog posts / newsletters
- [ ] `Organization` schema for org/project sites (Inclusion Geeks Library, etc.)
- [ ] Validate at: https://validator.schema.org

### Crawlability
- [ ] `robots.txt` — `Allow: /` + `Sitemap:` pointer
- [ ] `sitemap.xml` — list all public URLs
- [ ] Register in **Google Search Console** (free, shows indexing status + errors)
- [ ] Register in **Bing Webmaster Tools** (also free, covers Bing + DuckDuckGo)
- [ ] Consistent www vs. non-www — pick one and redirect the other (set in Cloudflare DNS)

### Content
- [ ] Single `<h1>` per page; logical heading order (h1 → h2 → h3)
- [ ] All `<img>` tags have descriptive `alt` text
- [ ] No broken internal links

---

## 2. Security headers

Set these at the server/CDN level — they can't be spoofed like meta tags can.

### `_headers` (Cloudflare Pages / Netlify)
```
/*
  X-Frame-Options: DENY
  X-Content-Type-Options: nosniff
  Referrer-Policy: strict-origin-when-cross-origin
  Permissions-Policy: camera=(), microphone=(), geolocation=(), payment=()
  Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
  Content-Security-Policy: default-src 'self'; script-src 'unsafe-inline'; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com; font-src https://fonts.gstatic.com; connect-src https://formspree.io; img-src 'self' data:; object-src 'none'; frame-ancestors 'none'
  X-Permitted-Cross-Domain-Policies: none
```

### `vercel.json` (Vercel)
```json
{
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        { "key": "X-Frame-Options", "value": "DENY" },
        { "key": "X-Content-Type-Options", "value": "nosniff" },
        { "key": "Referrer-Policy", "value": "strict-origin-when-cross-origin" },
        { "key": "Permissions-Policy", "value": "camera=(), microphone=(), geolocation=(), payment=()" },
        { "key": "Strict-Transport-Security", "value": "max-age=31536000; includeSubDomains; preload" },
        { "key": "Content-Security-Policy", "value": "default-src 'self'; script-src 'unsafe-inline'; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com; font-src https://fonts.gstatic.com; connect-src 'self'; img-src 'self' data:; object-src 'none'; frame-ancestors 'none'" },
        { "key": "X-Permitted-Cross-Domain-Policies", "value": "none" }
      ]
    }
  ]
}
```

### What each header does
| Header | Blocks |
|---|---|
| `X-Frame-Options: DENY` | Clickjacking (your site being embedded in an iframe) |
| `X-Content-Type-Options: nosniff` | MIME-type sniffing attacks |
| `Referrer-Policy` | Leaking the full URL of the page a visitor came from |
| `Permissions-Policy` | Pages requesting camera, mic, or location without your permission |
| `Strict-Transport-Security` | HTTP downgrade attacks; tells browsers to always use HTTPS |
| `Content-Security-Policy` | Injected scripts and styles from unauthorized origins |
| `frame-ancestors 'none'` | Same as X-Frame-Options but enforceable in modern browsers |

### Check your headers
Run your live URL through: https://securityheaders.com

---

## 3. Content Security Policy (CSP) — the nuance

The current CSP uses `script-src 'unsafe-inline'` because of the inline `<script>` in the contact form. This is the main limitation of static sites — CSP can't be fully locked down without a server generating a per-request **nonce**.

**If you ever add a build step** (Next.js, Astro, SvelteKit, etc.), you can tighten this significantly:
- Move all JS to external `.js` files → drop `'unsafe-inline'`
- Or add a nonce to inline scripts at render time

For now, the current CSP still meaningfully blocks:
- Scripts from any external domain you didn't list
- Iframes embedding your site
- Objects/embeds
- Data exfiltration to unlisted domains

---

## 4. HTML hygiene (applies to every site)

- [ ] All external links have `rel="noopener noreferrer"`
- [ ] All links use `https://` (never `http://`)
- [ ] No API keys, tokens, or secrets in client-side HTML/JS
- [ ] Honeypot field on any contact form (`<input type="text" name="_gotcha" style="display:none">`)
- [ ] `<html lang="en">` set correctly
- [ ] Favicon defined (avoids a wasted network request on every load)

---

## 5. Performance (Core Web Vitals affect SEO ranking)

Google uses these three signals as ranking factors.

### Largest Contentful Paint (LCP) — load speed
- [ ] `<link rel="preload" as="image" href="hero.jpg">` for the above-fold image
- [ ] `fetchpriority="high" loading="eager"` on the LCP `<img>`
- [ ] `loading="lazy"` on everything below the fold
- [ ] `width` and `height` on every `<img>` (lets browser reserve space before image loads)
- [ ] Serve images in **WebP** format (30–50% smaller than JPEG/PNG)
- [ ] Images sized for their display size (don't serve a 3000px image in a 400px slot)

### Cumulative Layout Shift (CLS) — visual stability
- [ ] `width`/`height` on all images and videos
- [ ] No content injected above existing content after load
- [ ] Font fallbacks match web font metrics closely (`font-display: swap` in Google Fonts ✅)

### Interaction to Next Paint (INP) — responsiveness
- [ ] Avoid large JS bundles blocking the main thread
- [ ] Debounce heavy event handlers

### Tools
- **PageSpeed Insights**: https://pagespeed.web.dev (paste your URL, get a score)
- **WebPageTest**: https://www.webpagetest.org (more detailed)

---

## 6. Dynamic sites (Railway + Supabase + APIs)

These apply to any site that has a backend — `didwestartawartoday.com`, `globalprotesttracker.com`, etc.

### API security
- [ ] **Never expose secret keys in client-side code** — use environment variables server-side
- [ ] Rate-limit API endpoints (Railway: use a middleware like `express-rate-limit`)
- [ ] Validate and sanitize all user input before it touches a database
- [ ] Use parameterized queries / ORM methods — never string-concatenate SQL
- [ ] Return generic error messages to clients (don't leak stack traces)
- [ ] Set `CORS` to only allow your own frontend domain, not `*`

### Supabase specifically
- [ ] Row-Level Security (RLS) enabled on all tables
- [ ] Never use the `service_role` key in the browser — only `anon` key
- [ ] Review which tables/columns are exposed via the auto-generated REST API
- [ ] Rotate keys if they've ever been committed to git

### Environment variables
- [ ] `.env` in `.gitignore` — check with `git status` before every commit
- [ ] Use Railway's environment variable dashboard, not hardcoded values
- [ ] Audit your public GitHub repos for accidentally committed secrets:
  https://github.com/settings/security-analysis (GitHub Secret Scanning)

### Authentication (if/when you add it)
- [ ] Use an established provider (Supabase Auth, Clerk, Auth.js) — don't roll your own
- [ ] Cookies: `HttpOnly`, `Secure`, `SameSite=Lax`
- [ ] Implement CSRF protection for any state-changing form submissions
- [ ] Short-lived JWTs with refresh tokens

---

## 7. Cloudflare settings to enable

If your domain is proxied through Cloudflare (orange cloud), these are free and worth turning on:

- [ ] **SSL/TLS mode: Full (strict)** — encrypts all the way to your origin
- [ ] **Always Use HTTPS** — redirects HTTP → HTTPS automatically
- [ ] **HSTS** — enable in SSL settings (Cloudflare can do this for you)
- [ ] **Email Address Obfuscation** — hides email addresses from scrapers
- [ ] **Hotlink Protection** — prevents other sites from embedding your images
- [ ] **Bot Fight Mode** — free basic bot blocking
- [ ] **Security Level: Medium** — blocks known bad IPs

---

## 8. Privacy & compliance

- [ ] If you collect **any** personal data (email via contact form counts): have a privacy policy
- [ ] If you use **analytics** (Google Analytics, Plausible, etc.): disclose it; consider a cookie banner for EU visitors
- [ ] Formspree stores submitted data — check their retention policy and mention it in your privacy policy
- [ ] **GDPR**: if you have EU visitors, you need a legal basis for processing their data and a way for them to request deletion

---

## 9. Maintenance

- [ ] Run `npm audit` on any project with a `package.json` periodically
- [ ] Check [securityheaders.com](https://securityheaders.com) after any infrastructure change
- [ ] Check [Google Search Console](https://search.google.com/search-console) monthly for crawl errors
- [ ] Renew SSL certificates — Cloudflare and Vercel auto-renew, but verify
- [ ] Review Formspree submissions for spam (if honeypot isn't catching everything, add reCAPTCHA)

---

## remtheory.com status

| Item | Status |
|---|---|
| Meta description | ✅ |
| Open Graph / Twitter Card | ✅ |
| Canonical URL | ✅ |
| JSON-LD Person schema | ✅ |
| robots.txt + sitemap.xml | ✅ |
| Security headers (`_headers`) | ✅ |
| CSP + Referrer-Policy meta fallbacks | ✅ |
| HTTPS links only | ✅ |
| `noopener noreferrer` on all external links | ✅ |
| Honeypot on contact form | ✅ |
| LCP image preloaded | ✅ |
| `loading="lazy"` on below-fold images | ✅ |
| `width`/`height` on images | ✅ |
| Favicon | ✅ |
| Google Search Console registration | ⬜ Do manually |
| WebP images | ⬜ Nice to have |
| Privacy policy | ⬜ Worth adding |

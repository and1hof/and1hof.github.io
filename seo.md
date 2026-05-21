# SEO Backlog

Outstanding SEO issues identified in the audit. Items numbered to match the original audit; completed items (#1, #3, #5, #7, #8, #9, #10, #11, #13) are not included here.

---

## High-impact

### #2 — Mobile viewport blocks pinch-zoom
[_includes/meta.html](_includes/meta.html) currently uses `width=device-width, initial-scale=1` (already fixed in this round as part of the meta.html rewrite — verify and remove from backlog if confirmed).

**Status:** likely already resolved by the meta.html overhaul. Confirm in a build before closing.

### #4 — `og:image` is the avatar on every page
[_layouts/default.html](_layouts/default.html) hardcodes `assets/avatar.jpg`. Posts with hero images (Trusted Types, SCA Reachability, etc.) lose social-share CTR.

**Fix:** replace the hardcoded line with:
```html
<meta property="og:image" content="{% if page.image %}{{ site.url }}{{ page.image }}{% else %}{{ site.url }}{{ site.avatar }}{% endif %}" />
```
Per-post `image:` front-matter values are already added (round 1 of #10/#13). Wire-up is a one-line edit.

Update `twitter:image` in `_includes/meta.html` the same way.

### #6 — Site description is generic
[_config.yml:9](_config.yml#L9) — "Software Engineering & Application Security" is hobby-level copy for a consulting business. Ripples into every `<title>`, OG fallback, and RSS feed.

**Candidates:**
- "Application Security Consulting, Books & Research"
- "Web Application Security Consulting — Andrew Hoffman, LLC"
- "AppSec Consulting · Author of *Web Application Security* (O'Reilly)"

### #12 — Amazon affiliate links lack `rel="sponsored"`
Amazon Associates TOS + Google both require disclosure. Affected links:
- [_layouts/default.html:50](_layouts/default.html#L50) — `book` nav link
- [llc.md](llc.md) — 2nd Edition + 1st Edition buttons
- [consulting.md](consulting.md) — book mentions

**Fix:** `rel="sponsored noopener noreferrer"` on every `amzn.to` link.

---

## Medium-impact

### #14 — No image optimization pipeline
Unoptimized PNG/JPG in [assets/](assets/) hurt LCP (a Core Web Vitals signal).

**Options:**
- Run all assets through `sharp-cli` or `imagemin`, commit compressed versions
- Add `<picture>` with WebP sources around hero images
- Add `loading="lazy"` and `decoding="async"` to non-LCP images

### #15 — Blog is stale
Last post: December 2022. Even with great backlinks, Google demotes domains with no fresh content.

**Action:** publish at minimum quarterly. Use `last_modified_at:` in front matter when tweaking posts and surface via sitemap `lastmod`.

### #16 — No per-page `seo_title` override
[_layouts/default.html:4](_layouts/default.html#L4) title template has no override hook. Some posts would benefit from a different SEO title than display title.

**Fix:** support `page.seo_title` with fallback to `page.title`:
```liquid
<title>{% if page.seo_title %}{{ page.seo_title }}{% elsif page.title %}{{ page.title }} – {{ site.name }}{% else %}{{ site.name }} – {{ site.description }}{% endif %}</title>
```

### #17 — No breadcrumb schema
Posts have no `BreadcrumbList` JSON-LD. Adding it (Home → Blog → Post Title) gives rich-snippet eligibility and improves SERP CTR.

Add to [_layouts/post.html](_layouts/post.html).

### #18 — `/llc/` page is thin on content
~80 words of body copy for the LLC hub. Underweight for ranking on "andrew hoffman llc", "appsec consulting", etc.

**Action:** add 200–400 words below the cards covering what the LLC does, who it serves, geographic/remote reach, credentials, and a CTA.

### #19 — No per-post `<link rel="alternate">` for RSS
Site-wide feed is linked, but per-post atom alternates aren't. Minor.

### #20 — Tags exist in CSS but not in content
[style.scss:287](style.scss#L287) styles `.tag` but no posts use tags, and no `/tags/` index exists. Tag pages create internal-link hubs.

**Action:** add tags to every post (`tags: [xss, csp, browser-security]`) and build a `/tags/{tag}/` template.

### #21 — Heading hierarchy in posts skips H2
Most posts jump from `<h1>` (title) directly to `### Background` (H3). Fix to maintain hierarchical structure.

**Action:** convert top-level section headings in posts from `###` to `##`.

### #22 — `_site/` is being committed with localhost URLs
[_site/sitemap.xml](_site/sitemap.xml) and [_site/robots.txt](_site/robots.txt) reference `http://localhost:4000`. If `_site/` is being deployed, Google sees localhost URLs.

**Fix:** add `_site/` to `.gitignore`; verify GitHub Pages rebuilds from source.

### #23 — Missing `theme-color` and favicon variants
- No `<meta name="theme-color">` for mobile browser chrome
- Only `favicon.ico` — no `apple-touch-icon`, no `manifest.json`, no `mask-icon.svg`

### #24 — `book` nav link bypasses your own catalog
[_layouts/default.html:50](_layouts/default.html#L50) sends visitors straight to Amazon. Consider a `/book/` landing page on your own domain that ranks for "andrew hoffman web application security book", and have *that* link to Amazon.

---

## Low-impact / polish

### #25 — Permalink fragility
`permalink: /:title/` would silently collide on duplicate slugs. Consider `/:year/:title/` for blog and reserve root-level slugs for landing pages.

### #26 — No author bio block at end of posts
Add a footer card to [_layouts/post.html](_layouts/post.html): photo + 2 sentences + "Hire Andrew" CTA → `/llc/`. Internal-link equity + conversion.

### #27 — No "Related posts" section
With only 9 posts, a simple by-tag related-list is trivial and creates internal links.

### #28 — No analytics or Search Console verification
- Add Google Search Console + verification meta
- Add privacy-friendly analytics (Plausible, Fathom, or GA4)
- Can't improve what you can't measure

### #29 — Empty footer-links in config
[_config.yml:19-32](_config.yml#L19-L32) — prune empty fields and add Amazon author profile + O'Reilly profile URLs to Person schema `sameAs` ([about.md](about.md)).

### #30 — Post titles use `&#58;` for colons
Cleaner to quote: `title: "Trusted Types: Future-proof XSS Defense"` (already fixed for posts touched in #10).

**Remaining:** verify no other posts still use `&#58;` (rapid grep).

---

## Recommended next round

If picking the next batch, do:
1. **#4** (per-post og:image) — 1-line edit, unlocks better social previews on every blog share
2. **#6** (site description rewrite) — 1-config-line edit, ripples through every title tag and feed
3. **#12** (`rel="sponsored"`) — small but Amazon TOS / Google compliance
4. **#21** (h3 → h2 in posts) — bulk edit but high accessibility + SEO value
5. **#28** (Search Console + analytics) — you need this before measuring whether anything else worked

The rest can be queued for subsequent cycles.

---

# Performance Backlog

Items from the performance audit not yet implemented. Numbers match the original performance audit (separate from SEO numbering above).

## Deferred this round

### Perf #5 — SVG icons CSS bloat (53.8KB → ~10KB possible)
[_sass/_svg-icons.scss](_sass/_svg-icons.scss) ships 13 base64-encoded SVG icons; only 5 render in the footer (github, linkedin, rss, twitter, youtube). Base64 adds ~33% overhead and the entire blob is in render-blocking CSS.

**Status:** Deferred — user wants to keep icons as-is for now. Revisit when ready to reduce CSS payload further.

**Recommended fix when revisiting:**
- Prune the 8 unused icon definitions
- Convert base64 to URL-encoded SVG (smaller raw + better gzip)
- Run SVGO on each icon's markup
- Expected: 53.8KB → ~10–15KB raw, no behavior change

### Perf #11 — SVG favicon
[favicon.ico](favicon.ico) is a 16KB ICO. Modern browsers all support SVG favicons at ~1KB.

**Status:** Deferred — user to provide an SVG file. Add `<link rel="icon" href="/favicon.svg" type="image/svg+xml">` to [_layouts/default.html](_layouts/default.html) once the asset is in place; keep the existing ICO as `<link rel="alternate icon">` for legacy browsers.

## Outstanding (not yet attempted)

### Perf #1 — Homepage image-bomb
[index.html](index.html) renders `{{ post.excerpt }}` per post; Jekyll's default excerpt is the first paragraph which contains the hero `<img>`. Loads ~2 MB of images on the homepage.

**Fix:** strip images from excerpts. Replace `{{ post.excerpt }}` with:
```liquid
{{ post.content | split: '</p>' | first | append: '</p>' | strip_html | truncatewords: 40 }}...
```
or use `excerpt_separator` + manual `<!--more-->` markers in posts.

### Perf #2, #3, #4, #7, #9, #12, #13, #14, #17 — DONE this round
WebP/picture fallback, lazy loading, width/height on every img, 300px hero cap, dead-asset cleanup, avatar fetchpriority, preload hints, LLC CSS extraction, .llc-btn transition removal — all implemented.

### Perf #6 — Critical CSS inlining
Inline above-the-fold CSS in `<head>`, defer the rest via preload + onload swap. Worth ~200–400 ms LCP on slow connections.

### Perf #8 — Syntax-highlighting CSS shipped everywhere
[_sass/_highlights.scss](_sass/_highlights.scss) is loaded on every page but only used on posts with code blocks. Scope it or split into a post-only stylesheet.

### Perf #10 — DNS prefetch / preconnect for Amazon
Add to `<head>`:
```html
<link rel="dns-prefetch" href="//amzn.to">
<link rel="dns-prefetch" href="//www.amazon.com">
```

### Perf #15 — `srcset` for HiDPI screens
Particularly for the avatar (served at 200×200, displayed at 70×70 — already retina-friendly) and any image displayed at less than its natural resolution.

### Perf #16 — Already done (`sass: style: :compressed`)

### Perf #18 — Cache headers / immutable assets
GitHub Pages-side; out of scope for the source tree.

---
title: "SEO Optimizations for a Hugo Site on Cloudflare"
date: 2026-03-29
lastmod: 2026-03-29
tags: ["hugo", "seo", "cloudflare", "blowfish"]
categories: ["tech"]
summary: "A practical checklist for SEO on a Hugo site with the Blowfish theme hosted on Cloudflare — from meta tags and structured data to Google Search Console and Cloudflare caching."
---

After getting [the blog live](/tech/how-i-built-this-blog/), the next step is making sure search engines can find it. Hugo and Blowfish already do a lot of the heavy lifting — but there are gaps in the default config that leave SEO value on the table.

Here is everything I configured, why it matters, and how to verify it works.

## What Blowfish Gives You for Free

Before changing anything, the Blowfish theme already provides:

| Feature | Status |
|---|---|
| `<meta name="description">` from page summary | Automatic |
| `<meta name="keywords">` from page tags | Automatic |
| `<link rel="canonical">` on every page | Automatic |
| Open Graph tags (`og:title`, `og:description`, `og:image`) | Automatic |
| Twitter Card tags | Automatic |
| JSON-LD structured data (`WebSite` on homepage, `Article` on posts) | Automatic |
| RSS feed at `/index.xml` | Automatic |
| Sitemap at `/sitemap.xml` | Automatic |
| `robots.txt` (when `enableRobotsTXT = true`) | Automatic |
| `rel="me"` links for author social profiles | Automatic |

This is a solid baseline. The work below fills in the gaps.

## 1. Configure Author Information

Without author info, the JSON-LD schema has an empty `author.name`, the `<meta name="author">` tag is blank, and RSS feeds have no author. Search engines use this to connect content to a person.

Add to `config/_default/languages.en.toml`:

```toml
[params.author]
  name = "Nikhil Joshi"
  headline = "Software engineer writing about technology and management."
  links = [
    { linkedin = "https://www.linkedin.com/in/njoshi/" },
    { github = "https://github.com/nnjoshi14" },
  ]
```

This populates the author field across meta tags, JSON-LD, RSS, and Open Graph.

## 2. Add a Default Social Sharing Image

When someone shares a link on LinkedIn, Twitter, or Slack, the platform looks for an `og:image`. Without one, your link shows with a blank preview — easy to scroll past.

Create a 1200x630px image (the optimal size for most platforms) and save it as `assets/img/social-banner.svg`. Then add to `config/_default/params.toml`:

```toml
defaultSocialImage = "img/social-banner.svg"
```

Blowfish uses a fallback chain for images: it first looks for a page-level image named `featured*`, `cover*`, or `thumbnail*`, then falls back to this default. So individual posts can have their own social images while the site-wide default catches everything else.

## 3. Enable Structured Breadcrumbs

Blowfish includes a `BreadcrumbList` JSON-LD schema, but it is off by default. Breadcrumbs give Google additional context about your site hierarchy and can appear as rich results in search.

Add to `config/_default/params.toml`:

```toml
enableStructuredBreadcrumbs = true
```

This generates structured data like:

```json
{
  "@type": "BreadcrumbList",
  "itemListElement": [
    { "name": "Home", "item": "https://builtbynikhil.com/" },
    { "name": "Tech", "item": "https://builtbynikhil.com/tech/" },
    { "name": "SEO Optimizations", "item": "https://builtbynikhil.com/tech/seo-for-hugo-cloudflare/" }
  ]
}
```

## 4. Register with Google Search Console

Google does not know your site exists until you tell it. [Google Search Console](https://search.google.com/search-console) lets you submit your sitemap, monitor indexing, and see what queries bring traffic.

1. Go to Google Search Console and add `https://builtbynikhil.com` as a property
2. Choose **Domain** property type (covers the entire domain including subdomains)
3. Choose **DNS record** verification
4. Copy the TXT record value Google provides
5. In the [Cloudflare Dashboard](https://dash.cloudflare.com), go to your domain's **DNS** settings and add a TXT record:
   - **Name:** `@`
   - **Content:** the value from Google (e.g., `google-site-verification=abc123...`)
6. Go back to Search Console and click **Verify** — DNS propagation on Cloudflare is near-instant
7. Submit your sitemap URL: `https://builtbynikhil.com/sitemap.xml`

DNS verification is simpler than the HTML meta tag approach — it does not depend on your site being deployed or the theme rendering correctly, and it works for the entire domain in one step.

Google will start crawling and indexing your pages. It takes a few days to see data.

## 5. Register with Bing Webmaster Tools

Bing powers search results for Bing, Yahoo, DuckDuckGo (partially), and other services. The setup is similar:

1. Go to [Bing Webmaster Tools](https://www.bing.com/webmasters)
2. Add your site and choose **CNAME record** verification (or import directly from Google Search Console — Bing offers this shortcut)
3. Add the DNS record in Cloudflare if using CNAME verification
4. Verify, then submit your sitemap: `https://builtbynikhil.com/sitemap.xml`

## 6. Include Taxonomy Pages in the Sitemap

By default, Blowfish excludes tag and category pages from the sitemap:

```toml
[sitemap]
  excludedKinds = ["taxonomy", "term"]
```

Tag pages like `/tags/hugo/` are valid landing pages — someone searching for "hugo blogging" could land there. Include them by overriding this in `config/_default/params.toml`:

```toml
[sitemap]
  excludedKinds = []
```

## 7. Add Social Sharing Links to Articles

Make it easy for readers to share your posts. Add to the `[article]` section in `config/_default/params.toml`:

```toml
[article]
  sharingLinks = ["linkedin", "twitter", "bluesky", "reddit", "email"]
```

Sharing generates backlinks and social signals, both of which help SEO indirectly.

## 8. Cloudflare Performance Settings

Site speed is a Google ranking factor. Cloudflare gives you several free optimizations — enable these in the Cloudflare dashboard:

| Setting | Where | What It Does |
|---|---|---|
| **Brotli compression** | Speed > Optimization | Compresses text assets ~20% better than gzip |
| **Early Hints (103)** | Speed > Optimization | Preloads critical assets before HTML response |
| **HTTP/2 and HTTP/3** | Network | Multiplexed connections, faster page loads |
| **Always Use HTTPS** | SSL/TLS > Edge Certificates | Redirects HTTP to HTTPS (ranking signal) |
| **HSTS** | SSL/TLS > Edge Certificates | Tells browsers to always use HTTPS |
| **Browser Cache TTL** | Caching > Configuration | Set to 4+ hours for static assets |

Leave **Rocket Loader** off — it can interfere with Hugo's already-optimized JavaScript loading.

## 9. Add Security Headers

Security headers do not directly affect SEO rankings, but Google has indicated that site security is a trust signal. Add a `static/_headers` file:

```
/*
  X-Content-Type-Options: nosniff
  X-Frame-Options: DENY
  Referrer-Policy: strict-origin-when-cross-origin
```

## 10. Write Good Frontmatter on Every Post

All the meta tags in the world are useless if the content behind them is empty. Every post should have:

```yaml
---
title: "Descriptive, keyword-rich title"
date: 2026-03-29
tags: ["relevant", "specific", "tags"]
categories: ["tech"]
summary: "A 1-2 sentence description that reads well in search results."
---
```

The `summary` field is particularly important — Blowfish uses it as the `<meta name="description">`, which is what appears under your title in Google results. Keep it under 160 characters for the best display.

For updated posts, add `lastmod` to signal freshness:

```yaml
lastmod: 2026-04-15
```

This updates the `dateModified` in JSON-LD and `lastmod` in the sitemap — both signals that search engines use to prioritize crawling.

## Verification Checklist

After making these changes, verify everything works:

**Meta tags** — view page source and check for:
- `<meta name="description" content="...">`
- `<meta name="author" content="Nikhil Joshi">`
- `<meta name="keywords" content="...">`
- `<link rel="canonical" href="...">`

**Open Graph** — paste your URL into the [Facebook Sharing Debugger](https://developers.facebook.com/tools/debug/) or [LinkedIn Post Inspector](https://www.linkedin.com/post-inspector/) and confirm title, description, and image appear.

**Structured data** — paste your URL into [Google's Rich Results Test](https://search.google.com/test/rich-results) and confirm `Article`, `WebSite`, and `BreadcrumbList` schemas are valid.

**Sitemap** — visit `https://builtbynikhil.com/sitemap.xml` and confirm all pages are listed.

**Robots.txt** — visit `https://builtbynikhil.com/robots.txt` and confirm it says `Allow: /` with a `Sitemap:` directive.

**Page speed** — run [PageSpeed Insights](https://pagespeed.web.dev/) and aim for 90+ on both mobile and desktop. A static Hugo site on Cloudflare should score very high with minimal effort.

## Summary of Config Changes

Here is everything in one place:

**`config/_default/languages.en.toml`** — add:

```toml
[params.author]
  name = "Nikhil Joshi"
  headline = "Software engineer writing about technology and management."
  links = [
    { linkedin = "https://www.linkedin.com/in/njoshi/" },
    { github = "https://github.com/nnjoshi14" },
  ]
```

**`config/_default/params.toml`** — add or update:

```toml
enableStructuredBreadcrumbs = true
defaultSocialImage = "img/social-banner.svg"

[article]
  sharingLinks = ["linkedin", "twitter", "bluesky", "reddit", "email"]

[sitemap]
  excludedKinds = []
```

Search engine verification is done via DNS records in Cloudflare — no config changes needed in Hugo.

The full source for this site is on GitHub: [nnjoshi14/builtbynikhil](https://github.com/nnjoshi14/builtbynikhil).

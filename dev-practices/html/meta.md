# Meta Tags, SEO & Structured Data

## Essential `<head>` Tags

```html
<head>
  <!-- Character encoding — first tag, within first 1024 bytes -->
  <meta charset="UTF-8">

  <!-- Responsive viewport -->
  <meta name="viewport" content="width=device-width, initial-scale=1.0">

  <!-- Page title — shown in browser tab, search results, and link previews -->
  <title>Page Title – Site Name</title>

  <!-- Short description — shown in search snippets (150–160 characters) -->
  <meta name="description" content="A concise description of this page's content.">

  <!-- Canonical URL — prevents duplicate content issues -->
  <link rel="canonical" href="https://example.com/page/">

  <!-- Language -->
  <html lang="en">
</head>
```

- `charset` must appear within the first 1024 bytes — place it first
- Title format: `{Page Name} – {Site Name}` (use en-dash `–`, not hyphen)
- Keep titles under 60 characters to avoid truncation in search results
- `<meta name="description">` does not directly affect ranking but influences click-through rate
- Do **not** use `<meta name="keywords">` — search engines ignore it

## Robots Directives

```html
<!-- Allow indexing (default behavior — no tag needed) -->
<meta name="robots" content="index, follow">

<!-- Prevent indexing this page -->
<meta name="robots" content="noindex, nofollow">

<!-- Allow indexing, prevent following links -->
<meta name="robots" content="index, nofollow">
```

Prefer a `robots.txt` file for site-wide rules; use the meta tag for per-page exceptions.

## Open Graph (Social Sharing)

```html
<meta property="og:type" content="website">
<meta property="og:title" content="Page Title">
<meta property="og:description" content="Description shown in link previews.">
<meta property="og:url" content="https://example.com/page/">
<meta property="og:image" content="https://example.com/og-image.jpg">
<meta property="og:image:width" content="1200">
<meta property="og:image:height" content="630">
<meta property="og:site_name" content="Site Name">
<meta property="og:locale" content="en_US">
```

- Recommended OG image size: **1200×630 px** (1.91:1 ratio)
- Use an absolute URL for `og:image` — relative URLs are not supported
- `og:type` values: `website` (default), `article`, `product`, `video.movie`

## Twitter / X Card Tags

```html
<meta name="twitter:card" content="summary_large_image">
<meta name="twitter:title" content="Page Title">
<meta name="twitter:description" content="Description shown in X/Twitter cards.">
<meta name="twitter:image" content="https://example.com/twitter-image.jpg">
<meta name="twitter:site" content="@handle">
```

- `twitter:card` values: `summary`, `summary_large_image`, `app`, `player`
- If OG tags are present and Twitter tags are absent, Twitter falls back to OG values — you only need Twitter tags when the content differs

## Structured Data (JSON-LD)

JSON-LD is the recommended format (over Microdata or RDFa) — it is isolated in a `<script>` and does not affect HTML structure:

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": "How to Grow Tomatoes at Home",
  "author": {
    "@type": "Person",
    "name": "Alex Grower"
  },
  "datePublished": "2025-03-10",
  "dateModified": "2025-04-01",
  "image": "https://example.com/tomatoes.jpg",
  "publisher": {
    "@type": "Organization",
    "name": "Garden Weekly",
    "logo": {
      "@type": "ImageObject",
      "url": "https://example.com/logo.png"
    }
  }
}
</script>
```

Common Schema.org types:

| Type | Rich result |
|------|------------|
| `Article` / `NewsArticle` / `BlogPosting` | Article snippet |
| `Product` | Price, availability, ratings |
| `BreadcrumbList` | Breadcrumb path in SERP |
| `FAQPage` | Expandable FAQ in SERP |
| `Event` | Date, location, ticket info |
| `Organization` | Sitelinks search box, logo |
| `LocalBusiness` | Map pack info |

- Place JSON-LD in `<head>` or at the end of `<body>` — Google supports both
- Validate with [Google's Rich Results Test](https://search.google.com/test/rich-results)

## Favicon

```html
<!-- Modern approach — SVG favicon (scales perfectly, supports dark mode) -->
<link rel="icon" href="/favicon.svg" type="image/svg+xml">

<!-- PNG fallback for browsers without SVG icon support -->
<link rel="icon" href="/favicon-32.png" sizes="32x32" type="image/png">

<!-- Apple touch icon (180×180 PNG, shown on iOS home screen) -->
<link rel="apple-touch-icon" href="/apple-touch-icon.png">

<!-- Web app manifest (PWA) -->
<link rel="manifest" href="/site.webmanifest">
```

- A single 32×32 PNG favicon is the minimum viable set
- SVG favicons support `prefers-color-scheme` media queries for dark-mode variants
- No need for `favicon.ico` at the root if the `<link>` tags are present (browsers still check `/favicon.ico` as a fallback)

## Viewport

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```

- Do **not** add `user-scalable=no` or `maximum-scale=1` — this prevents users from zooming and fails WCAG 1.4.4
- Do **not** add `shrink-to-fit=no` (Safari legacy) without understanding the implications

## Alternate and Hreflang

```html
<!-- Hreflang for multilingual sites -->
<link rel="alternate" hreflang="en" href="https://example.com/en/page/">
<link rel="alternate" hreflang="fr" href="https://example.com/fr/page/">
<link rel="alternate" hreflang="x-default" href="https://example.com/page/">

<!-- RSS feed -->
<link rel="alternate" type="application/rss+xml" title="Site Feed" href="/feed.xml">
```

- Every language variant must include `hreflang` tags for all variants (including itself)
- `x-default` is the fallback URL when no other language matches

## Theme Color

```html
<!-- Browser UI color on mobile -->
<meta name="theme-color" content="#1a73e8">

<!-- Separate colors for light and dark mode -->
<meta name="theme-color" media="(prefers-color-scheme: light)" content="#ffffff">
<meta name="theme-color" media="(prefers-color-scheme: dark)" content="#1a1a2e">
```

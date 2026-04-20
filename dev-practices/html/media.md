# Media

## Responsive Images

### `srcset` + `sizes` — Raster Images at Multiple Resolutions

```html
<img
  src="hero-800.jpg"
  srcset="hero-400.jpg 400w, hero-800.jpg 800w, hero-1600.jpg 1600w"
  sizes="(max-width: 600px) 100vw, (max-width: 1200px) 50vw, 800px"
  alt="A mountain range at sunrise"
  width="800"
  height="450"
>
```

- `srcset` lists candidate files with their intrinsic widths (`w` descriptor)
- `sizes` tells the browser how wide the image will render at each viewport width
- Always include `width` and `height` attributes — the browser uses them to reserve layout space before the image loads, preventing Cumulative Layout Shift (CLS)
- The `src` is the fallback for browsers that do not understand `srcset`

### `<picture>` — Art Direction or Format Switching

```html
<picture>
  <!-- Modern format (AVIF first, then WebP) -->
  <source type="image/avif" srcset="photo.avif">
  <source type="image/webp" srcset="photo.webp">
  <!-- Fallback -->
  <img src="photo.jpg" alt="A coastal town at dusk" width="1200" height="800">
</picture>
```

```html
<!-- Art direction: different crop at different viewports -->
<picture>
  <source media="(max-width: 600px)" srcset="hero-portrait.jpg">
  <source media="(min-width: 601px)" srcset="hero-landscape.jpg">
  <img src="hero-landscape.jpg" alt="Team celebrating product launch" width="1200" height="675">
</picture>
```

- `<picture>` does not display anything itself — the `<img>` inside it is the rendered element and must always be present with a valid `alt`
- Sources are evaluated top to bottom; the first matching `<source>` wins

## Modern Image Formats

| Format | Savings vs JPEG | Transparency | Animation | Browser support |
|--------|----------------|--------------|-----------|----------------|
| WebP | ~30% | Yes | Yes | All modern browsers |
| AVIF | ~50% | Yes | Yes | Chrome, Firefox, Safari 16+ |
| JPEG XL | ~35–60% | Yes | Yes | Limited (Chrome flag only as of 2025) |

- Prefer AVIF for photos; fall back to WebP; always provide JPEG/PNG fallback via `<picture>`
- Use PNG for images requiring lossless quality or transparency where WebP support is a concern
- Use SVG for icons, logos, and illustrations — infinitely scalable, tiny file size

## Lazy Loading

```html
<!-- Native lazy loading — no JS required -->
<img src="below-fold.jpg" alt="..." loading="lazy" width="800" height="600">

<!-- Never lazy-load above-the-fold images (LCP impact) -->
<img src="hero.jpg" alt="..." loading="eager" width="1200" height="675">
```

- `loading="lazy"` defers off-screen images until the user scrolls near them
- Do **not** lazy-load the largest above-the-fold image — it directly harms Largest Contentful Paint (LCP)
- Use `loading="eager"` (or omit `loading`) for hero/LCP images
- Always set `width` and `height` alongside `loading="lazy"` to prevent layout shifts

## Preloading the LCP Image

```html
<link rel="preload" as="image" href="hero.jpg" fetchpriority="high">
```

Add this in `<head>` for the image that will be the LCP element.

## Video

```html
<video controls width="1280" height="720" preload="metadata" poster="thumbnail.jpg">
  <source src="intro.av1.mp4" type="video/mp4; codecs=av01">
  <source src="intro.webm" type="video/webm">
  <source src="intro.mp4" type="video/mp4">
  <track kind="subtitles" src="intro.en.vtt" srclang="en" label="English" default>
  <track kind="subtitles" src="intro.fr.vtt" srclang="fr" label="French">
  <p>Your browser does not support HTML video. <a href="intro.mp4">Download the video</a>.</p>
</video>
```

- `controls` is required for accessible playback — do not hide native controls without a custom equivalent
- `preload="metadata"` loads only duration/dimensions; avoids wasting bandwidth
- `poster` shows a thumbnail before play — prevents a blank black frame
- Captions (`<track kind="subtitles">` or `kind="captions"`) are required for WCAG 1.2.2 (AA)
- Captions (`kind="captions"`) include sound effects; subtitles (`kind="subtitles"`) are translations only
- Never autoplay video with sound — `autoplay` is acceptable only with `muted`

```html
<!-- Background / decorative video -->
<video autoplay muted loop playsinline aria-hidden="true">
  <source src="background.webm" type="video/webm">
  <source src="background.mp4" type="video/mp4">
</video>
```

- `aria-hidden="true"` removes decorative video from the accessibility tree
- `playsinline` prevents iOS from going full-screen automatically

## Audio

```html
<audio controls preload="metadata">
  <source src="podcast.opus" type="audio/ogg; codecs=opus">
  <source src="podcast.mp3" type="audio/mpeg">
  <p>Your browser does not support HTML audio. <a href="podcast.mp3">Download the audio</a>.</p>
</audio>
```

- Always provide `controls`
- Provide a transcript for audio content (WCAG 1.2.1)

## Aspect Ratios

Reserve image/video space before media loads to avoid CLS:

```html
<!-- Method 1: explicit width + height attributes (preferred) -->
<img src="photo.jpg" alt="..." width="16" height="9">
```

```css
/* CSS respects the intrinsic ratio from width/height attributes */
img {
  max-width: 100%;
  height: auto;
}
```

```css
/* Method 2: aspect-ratio property for containers */
.video-wrapper {
  aspect-ratio: 16 / 9;
  width: 100%;
}
```

## iframe Embeds

```html
<iframe
  src="https://www.youtube-nocookie.com/embed/VIDEO_ID"
  title="Demo of the new dashboard feature"
  width="560"
  height="315"
  loading="lazy"
  allowfullscreen
></iframe>
```

- Always provide `title` — it is the accessible name read by screen readers
- Use `loading="lazy"` for off-screen iframes
- Use `youtube-nocookie.com` to reduce tracking cookies

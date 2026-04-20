# Security

## XSS Prevention

Cross-site scripting (XSS) is the most common JavaScript security vulnerability.
The root cause is inserting untrusted data into the DOM as HTML.

### Never Use innerHTML With Untrusted Data

```js
// DANGEROUS — executes attacker-controlled scripts
container.innerHTML = userInput;
container.innerHTML = `<p>${commentText}</p>`;

// safe alternatives
container.textContent = userInput;          // text only, no HTML interpretation

// or build elements programmatically
const p = document.createElement('p');
p.textContent = commentText;
container.appendChild(p);
```

- Treat **all** data from URLs, query strings, localStorage, APIs, and user inputs as untrusted
- `textContent`, `createTextNode`, and `setAttribute` (for non-URL attributes) are safe for text

### Sanitizing HTML When You Must Render It

If you need to allow limited HTML (e.g., rich text), use a sanitizer — never write your own regex:

```js
// built-in browser sanitizer (Chrome 92+, Firefox 117+)
const sanitizer = new Sanitizer();
container.setHTML(userHtml, { sanitizer });

// or use DOMPurify (widely supported)
import DOMPurify from 'dompurify';
container.innerHTML = DOMPurify.sanitize(userHtml);
```

- Allowlist the tags and attributes you need — do not use blocklists
- Sanitize server-side too — defense in depth

### Dangerous APIs to Avoid

| API | Risk | Safe Alternative |
|-----|------|-----------------|
| `innerHTML =` | XSS | `textContent` or `setHTML` |
| `outerHTML =` | XSS | DOM methods |
| `document.write()` | XSS + blocks parsing | DOM methods |
| `eval()` | Code injection | JSON.parse, Function alternatives |
| `setTimeout(string)` | Code injection | `setTimeout(fn, delay)` |
| `new Function(string)` | Code injection | Avoid or CSP-restrict |

## Content Security Policy (CSP)

CSP is a browser mechanism that restricts what resources a page can load and execute.
Set it via HTTP header (preferred) or `<meta>` tag:

```
Content-Security-Policy:
  default-src 'self';
  script-src 'self' https://cdn.example.com;
  style-src 'self' 'unsafe-inline';
  img-src 'self' data: https:;
  connect-src 'self' https://api.example.com;
  frame-ancestors 'none';
```

- **Never use `'unsafe-inline'` for scripts** — it defeats XSS protection
- **Never use `'unsafe-eval'`** — it allows `eval()` and `new Function()`
- Use nonces or hashes for inline scripts that cannot be moved to external files:
  ```html
  <!-- server generates a fresh nonce per request -->
  <script nonce="abc123">...</script>
  ```
  ```
  Content-Security-Policy: script-src 'nonce-abc123'
  ```
- Start with a report-only policy (`Content-Security-Policy-Report-Only`) and monitor violations before enforcing

## postMessage Validation

Always validate the origin and message shape when receiving `postMessage` events:

```js
window.addEventListener('message', (event) => {
  // always check origin
  if (event.origin !== 'https://trusted.example.com') return;

  // validate message structure before acting on it
  if (typeof event.data?.action !== 'string') return;

  handleMessage(event.data);
});
```

- Never use `event.origin === '*'` when sending sensitive data via `postMessage`
- Specify the target origin explicitly when sending:

```js
iframe.contentWindow.postMessage(payload, 'https://trusted.example.com');
// NOT: iframe.contentWindow.postMessage(payload, '*');
```

## Third-Party Scripts

- Load third-party scripts only from trusted CDNs and pin versions
- Use Subresource Integrity (SRI) to verify file contents haven't changed:

```html
<script
  src="https://cdn.example.com/lib.min.js"
  integrity="sha384-<hash>"
  crossorigin="anonymous">
</script>
```

- Isolate third-party scripts with a strict CSP
- Audit what data third-party scripts can access — they run with full page privileges
- Prefer self-hosting critical dependencies over loading from external CDNs

## URL and Redirect Safety

```js
// validate URLs before navigating or opening
function isSafeUrl(url) {
  try {
    const parsed = new URL(url);
    return ['https:', 'http:'].includes(parsed.protocol);
  } catch {
    return false;
  }
}

// bad — allows javascript: URLs
location.href = userProvidedUrl;

// good
if (isSafeUrl(userProvidedUrl)) {
  location.href = userProvidedUrl;
}
```

- Never pass user-controlled strings to `location.href`, `window.open`, or `<a href>` without validation
- Reject `javascript:`, `data:`, and `vbscript:` protocols explicitly

# Forms

## Basic Structure

```html
<form action="/submit" method="post" novalidate>
  <fieldset>
    <legend>Shipping Address</legend>
    <div>
      <label for="street">Street</label>
      <input type="text" id="street" name="street" autocomplete="street-address" required>
    </div>
  </fieldset>
  <button type="submit">Submit</button>
</form>
```

- Always specify `method` — default is `GET`, which exposes data in the URL
- Use `novalidate` on the `<form>` when implementing custom validation UI (still run your own validation)
- Wrap related fields in `<fieldset>` with a `<legend>` for grouped context

## Input Types

Use the correct `type` — browsers provide native UI, keyboard optimization, and built-in validation:

| Type | Use for |
|------|---------|
| `text` | General short text |
| `email` | Email addresses |
| `tel` | Phone numbers |
| `url` | Web addresses |
| `number` | Numeric values with optional `min`, `max`, `step` |
| `range` | Sliders (numeric range) |
| `date` / `time` / `datetime-local` | Dates and times |
| `password` | Sensitive text (masked) |
| `search` | Search boxes |
| `checkbox` | Binary on/off choice |
| `radio` | One-of-many choice (group by shared `name`) |
| `file` | File upload |
| `hidden` | Values not shown to user |
| `color` | Color picker |

- Do **not** use `type="text"` for emails, URLs, or phone numbers — lose free validation and mobile keyboard hints
- `type="number"` is for math; use `type="text" inputmode="numeric" pattern="[0-9]*"` for things like zip codes

## Labels

Every input **must** have an associated label:

```html
<!-- Preferred: explicit association via for/id -->
<label for="email">Email address</label>
<input type="email" id="email" name="email">

<!-- Acceptable: wrapping label -->
<label>
  Email address
  <input type="email" name="email">
</label>
```

- Never use `placeholder` as a substitute for a `<label>` — it disappears on focus and has low contrast
- Placeholder text is acceptable as a hint in addition to a label
- For icon-only buttons or inputs: use `aria-label` or visually hidden text

## Built-in Validation

HTML5 validation attributes — use them; they are free:

```html
<input type="email" required>
<input type="text" minlength="2" maxlength="50">
<input type="number" min="1" max="100" step="1">
<input type="text" pattern="[A-Z]{2}[0-9]{4}" title="Two letters followed by four digits">
<input type="url" required>
```

- `required` — field must have a value
- `minlength` / `maxlength` — character count bounds
- `min` / `max` — numeric/date bounds
- `pattern` — regex (applied to the full value; no need to anchor with `^$`)
- `title` on `pattern` inputs describes the expected format to users

## Custom Validation

When built-in messages are insufficient, use the Constraint Validation API:

```html
<input type="text" id="username" name="username" required aria-describedby="username-error">
<span id="username-error" role="alert" hidden></span>
```

```js
const input = document.getElementById('username');
const error = document.getElementById('username-error');

input.addEventListener('blur', () => {
  if (!input.validity.valid) {
    error.textContent = 'Username is required.';
    error.hidden = false;
    input.setAttribute('aria-invalid', 'true');
  } else {
    error.hidden = true;
    input.removeAttribute('aria-invalid');
  }
});
```

- Associate error messages with `aria-describedby`
- Use `aria-invalid="true"` on invalid inputs
- Use `role="alert"` on error containers so screen readers announce them immediately
- Show errors on `blur` (field left) or form submit — not on every keystroke

## Fieldset and Legend

```html
<fieldset>
  <legend>Preferred contact method</legend>
  <label><input type="radio" name="contact" value="email"> Email</label>
  <label><input type="radio" name="contact" value="phone"> Phone</label>
</fieldset>
```

- Required for groups of `radio` or `checkbox` inputs — the `<legend>` provides shared context
- Optional but useful for any logically grouped set of fields (e.g., billing address)
- The `<legend>` must be the **first child** of `<fieldset>`

## Autocomplete

Always set `autocomplete` on personal data fields — improves UX and is required by WCAG 1.3.5:

```html
<input type="text"  name="name"     autocomplete="name">
<input type="email" name="email"    autocomplete="email">
<input type="tel"   name="phone"    autocomplete="tel">
<input type="text"  name="address"  autocomplete="street-address">
<input type="text"  name="city"     autocomplete="address-level2">
<input type="text"  name="zip"      autocomplete="postal-code">
<input type="text"  name="cc"       autocomplete="cc-number">
```

Use `autocomplete="off"` only for truly sensitive, one-time fields (e.g., OTP, CAPTCHA).

## Buttons

```html
<!-- Inside a form — submits it -->
<button type="submit">Place Order</button>

<!-- Inside a form — does not submit -->
<button type="button" id="add-item">Add Item</button>

<!-- Resets form fields -->
<button type="reset">Clear</button>
```

- Always specify `type` on `<button>` — the default is `submit`, which surprises many developers
- Use `<button>`, not `<input type="submit">` — `<button>` can contain HTML
- Never disable a submit button to prevent submission; show validation errors instead

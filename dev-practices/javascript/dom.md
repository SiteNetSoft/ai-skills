# DOM — Querying, Events & Mutations

## Querying the DOM

- Prefer `querySelector` / `querySelectorAll` over older `getElementById` / `getElementsByClassName`
- Cache query results — do not query the DOM in loops or on every event
- Scope queries to a parent element when possible to avoid searching the whole document:

```js
const form = document.querySelector('#signup-form');
const inputs = form.querySelectorAll('input[required]');
```

- `querySelectorAll` returns a static `NodeList` — spread or `Array.from()` to use array methods:

```js
const values = [...form.querySelectorAll('input')].map(el => el.value);
```

## Event Handling

### Prefer addEventListener Over Inline Handlers

```js
// good
button.addEventListener('click', handleClick);

// bad — pollutes global scope, can't add multiple handlers
button.onclick = handleClick;
```

### Event Delegation

Attach one listener to a common ancestor instead of many listeners on individual elements.
Especially important for dynamic lists:

```js
document.querySelector('#todo-list').addEventListener('click', (e) => {
  const item = e.target.closest('.todo-item');
  if (!item) return;
  toggleItem(item.dataset.id);
});
```

### Cleanup with AbortController

Use `AbortController` to remove multiple listeners at once — no need to keep references to every handler:

```js
const controller = new AbortController();
const { signal } = controller;

element.addEventListener('click', handleClick, { signal });
element.addEventListener('keydown', handleKeydown, { signal });
window.addEventListener('resize', handleResize, { signal });

// remove all at once
controller.abort();
```

Ideal for component teardown, modal close, or route navigation.

### Listener Options

```js
// once — fires once then auto-removes
button.addEventListener('click', handler, { once: true });

// passive — tells browser the handler won't call preventDefault (scroll perf)
window.addEventListener('scroll', onScroll, { passive: true });

// capture — fires during capture phase instead of bubble
document.addEventListener('focus', trackFocus, { capture: true });
```

- Always use `{ passive: true }` for `scroll`, `touchstart`, `touchmove`, and `wheel` listeners

## Modifying the DOM

### Avoid Repeated DOM Writes

Batch DOM updates to minimize reflows. Use `DocumentFragment` to build subtrees off-screen:

```js
const fragment = document.createDocumentFragment();

items.forEach(item => {
  const li = document.createElement('li');
  li.textContent = item.label;   // textContent, not innerHTML
  fragment.appendChild(li);
});

list.appendChild(fragment);      // single reflow
```

- Use `textContent` to set text — never `innerHTML` with untrusted data (see security.md)
- Use `element.classList.add/remove/toggle/replace` instead of manipulating `className` strings
- Prefer CSS class toggling over inline style mutation for maintainability

### Reading and Writing Layout Properties

Never interleave reads and writes of layout properties — it causes layout thrashing:

```js
// bad — forces reflow on every iteration
elements.forEach(el => {
  const height = el.offsetHeight;    // read (forces layout)
  el.style.height = height + 10 + 'px'; // write
});

// good — batch reads, then batch writes
const heights = elements.map(el => el.offsetHeight);   // all reads
elements.forEach((el, i) => {
  el.style.height = heights[i] + 10 + 'px';             // all writes
});
```

## MutationObserver

Watch for DOM changes without polling:

```js
const observer = new MutationObserver((mutations) => {
  for (const mutation of mutations) {
    if (mutation.type === 'childList') {
      console.log('Children changed', mutation.addedNodes);
    }
  }
});

observer.observe(targetNode, {
  childList: true,
  subtree: true,
  attributes: false,
});

// always disconnect when done
observer.disconnect();
```

- Prefer `MutationObserver` over polling with `setInterval`
- Disconnect the observer when the component is destroyed to prevent memory leaks

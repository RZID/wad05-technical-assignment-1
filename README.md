# Technical Assignment 1 — Web Security & JS DOM Manipulation

## Contents

- `answer-q1-innerHTML-vs-textContent.md` — Q1: differences between
  `innerHTML` and `textContent`, when to use each, with examples.
- `answer-q2-xss-scenario.md` — Q2: real-world stored XSS scenario
  (vulnerability, exploitation, impact).
- `live-search.html` — Q3: standalone live search (no build step, no
  `innerHTML`).

## Q3 — How to run

No dependencies. Just open the file in a browser:

```bash
xdg-open live-search.html
```

Or with Python http server:

```bash
python3 -m http.server 8000
# open http://localhost:8000/live-search.html
```

Type in the search box — the list filters in real time. Implementation uses:

- `input` event for live updates
- `array.filter()` for filtering
- full re-render with `createElement` + `textContent` only

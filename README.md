# Technical Assignment 1 — Live Search (Q3)

Live search input that filters a dataset in real time. Answers for Q1 and Q2
are submitted as a separate PDF (not tracked in this repo).

## Contents

- `live-search.html` — search page (no build step, DOM built with
  `createElement` + `textContent` only)
- `data.json` — dataset (14 random items: makanan, minuman, elektronik,
  fashion, aksesoris)

## How to run

A local HTTP server is required because the page loads `data.json` via
`fetch` (browsers block `fetch` on `file://` URLs):

```bash
python3 -m http.server 8000
# open http://localhost:8000/live-search.html
```

Type in the search box — the list filters in real time. Implementation uses:

- `input` event for live updates
- `array.filter()` for filtering (by name and category)
- full re-render with `createElement` + `textContent` only

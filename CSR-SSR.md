# CSC13008 IA#3 — CSR vs SSR

## 1. Test Setup

- **Date:** 2026-11-02.
- **Executed by:** OpenAI Codex (VS Code), using installed Microsoft Edge **156.0.4314.58** in headless mode through Chrome DevTools Protocol **1.3**.
- **Server:** Existing `node server.mjs` server on localhost:5050; Node **v24.11.1**. Server mode was set to and confirmed as **normal**.
- **CSR:** `http://localhost:5050/?v=c`, using unchanged Version C.
- **SSR:** `http://localhost:5050/ssr`.
- **Conditions:** Browser extensions disabled; network cache disabled through CDP. JavaScript was disabled for Question 3, restored afterward, and enabled for the network comparison. No CPU or network throttling was applied.
- **Measurements:** Actual HTTP responses, both browser View Source pages, live DOM observations, network events, DOM mutation timestamps, and animation-frame layout checks. No mocked responses or new dependencies were used.

## 2. Question 1 — Page Source

> On `/?v=` any version and on `/ssr`, right-click → **View page source** (`Ctrl+U`). Where are the product names in each? Where is the stock value?

Both `view-source:http://localhost:5050/?v=c` and `view-source:http://localhost:5050/ssr` were opened in Edge, and their source lines were inspected alongside actual HTTP response bodies.

**CSR:** No product names, product prices, or calculated stock total occur in the initial HTML. The catalogue is empty:

```html
<ul class="catalogue" id="catalogue"></ul>
```

The stock output contains a placeholder:

```html
<output id="stock-value">—</output>
```

An inert `<template>` contains one empty card structure with blank `.name` and `.price` elements; this is not a rendered product. `panel.js` imports Version C, which fetches JSON and builds the cards in the browser. With JavaScript enabled, the live DOM later contained eight named/priced cards and `12.360.000 đ`.

**SSR:** The initial HTML already contains all eight `<li class="card" data-id="…">` elements, their names in `<h2 class="name">`, their formatted prices, and:

```html
<output id="stock-value">12.360.000 đ</output>
```

SSR therefore supplies pre-rendered product data. View Page Source shows the response HTML, whereas DevTools Elements shows the current DOM, including changes made by JavaScript; seeing cards in CSR's live DOM does not mean they were in its initial response.

## 3. Question 2 — HTML Escaping

> In `/ssr`'s source, find the scarf. Why does it read `&lt;i&gt;` there, and what does that have in common with `textContent` in the browser?

The actual SSR source contained:

```html
<h2 class="name">Scarf &lt;i&gt;limited&lt;/i&gt;</h2>
```

`escapeHtml` in `server.mjs:67–70` replaces `&`, `<`, `>`, double quotes, and apostrophes with HTML character references. `renderSsrPage` applies it to each product name at line 81 before inserting the result into HTML. The HTML parser decodes `&lt;` and `&gt;` into characters in a text node; it does not reinterpret those decoded characters as an `<i>` element.

CSR achieves the same text-only result through `card.querySelector('.name').textContent = p.name` in `public/versions/c.js:33`. That assignment creates text rather than parsing an HTML string. In both inspected live pages, the scarf read exactly `Scarf <i>limited</i>` and its name had zero `i`/`em` descendants. Both approaches prevent untrusted product names from being interpreted as HTML in these text locations.

## 4. Question 3 — JavaScript Disabled

> Disable JavaScript (DevTools settings, above) and reload both pages. What is left of each?

CDP's script-execution switch disabled page JavaScript, and both pages were reloaded before inspection.

| Observed item | CSR, JavaScript disabled | SSR, JavaScript disabled |
|---|---|---|
| Product cards | Zero | Eight |
| Names and prices | Absent | All eight names and formatted prices present |
| Stock total | `—` | `12.360.000 đ` |
| Status/loading line | Empty; no loading or error message | Empty |
| Page shell | Heading, navigation, test panel, and footer remain | Heading, navigation, catalogue, and footer remain |
| Controls | Five mode buttons remain visible, mode label stays `unknown`, version label stays blank, Retry remains hidden | No mode panel or Retry button is included in this page |

Clicking CSR's `slow` button with JavaScript disabled left the server in `normal` mode: its event handler was never installed. Ordinary navigation links remained in the HTML. No products or stock API requests were observed during the disabled-JavaScript loads.

CSR's empty catalogue is not a network failure: its initial HTML had no product data, and the loader never executed, so it could not fetch, render, or set a loading/error message. SSR's Node code had already produced the catalogue before the browser received it; disabling browser JavaScript does not disable server-side JavaScript. Browser JavaScript was restored after this test.

## 5. Question 4 — Network and Visibility

> In the Network tab, count the requests each page makes before the stock value appears. Which page would a search engine, or a user on a slow phone, see first?

With JavaScript enabled and cache disabled, each page was navigated in a separate browser target. Requests were counted from that navigation through the first animation-frame check observing the correct stock total with a visible layout box. Source-view requests, earlier disabled-JavaScript loads, and external setup requests were excluded.

| Comparison | CSR — Version C | SSR |
|---|---|---|
| Initial HTML content | Empty catalogue, empty card template, `—` total | Eight complete cards and `12.360.000 đ` |
| Browser products/stock API calls | 9: one products request and eight stock requests | 0 |
| Other browser data request | One `/__mode` request | None |
| Measured total requests before total appeared | **14** | **2** |
| JavaScript dependency | Required to fetch/render products and calculate the total | Not required for the catalogue or total |
| Product content available in DOM | **130.9 ms** after navigation time origin | **22.8 ms** |
| Correct total available in DOM | **982.5 ms** | **22.8 ms** |
| First frame checking layout-visible cards | **132.0 ms** | **56.3 ms** |

The actual request breakdown was:

| Resource category | CSR endpoints and count | SSR endpoints and count |
|---|---|---|
| HTML document | `/?v=c`: 1 | `/ssr`: 1 |
| CSS | `/style.css`: 1 | `/style.css`: 1 |
| Scripts | `/panel.js`, `/versions/c.js`: 2 | 0 |
| Mode panel | `/__mode`: 1 | 0 |
| Products | `/api/products`: 1 | 0 |
| Stock | `/api/stock/tshirt`, `/api/stock/hoodie`, `/api/stock/somi`, `/api/stock/cap`, `/api/stock/tote`, `/api/stock/socks`, `/api/stock/jacket`, `/api/stock/scarf`: 8 | 0 |
| Favicon or unrelated browser requests in the measured window | 0 | 0 |
| **Total** | **14** | **2** |

The disabled-JavaScript observations did include `/favicon.ico` requests, but neither measured JavaScript-enabled navigation window did. Favicon/browser request behaviour can vary with navigation history and browser state; the counts above are observed counts, not universal constants. No additional requests were observed during a 250 ms follow-up after each measurement cutoff.

**SSR made the catalogue available sooner in this local sample.** CSR showed a loading state, inserted its cards after the products response, and waited for the stock responses before displaying the total. SSR delivered cards and total together. Its server reads in-memory `PRODUCTS` and `STOCK` directly (`server.mjs:78–83`); it does not make the browser perform the nine API calls, nor does this SSR implementation make those nine HTTP calls internally.

For a crawler that does not execute JavaScript, the SSR response exposes product content immediately, while CSR exposes only the shell. Crawlers that execute JavaScript may also obtain CSR content, but must perform rendering and wait for the data; capabilities vary. On slow phones or connections, SSR can reduce the browser's initial script work and dependent data requests. These are implications of this implementation, not measured crawler or phone results, and not a guarantee that every SSR application is faster: server work, caching, network conditions, and infrastructure also matter.

## 6. Question 5 — When to Choose CSR or SSR

> One sentence: when would you want the server to render, and when the browser?

Choose SSR for immediately available, indexable initial content and CSR for highly interactive experiences that benefit from browser-side updates, while evaluating performance and SEO against the actual implementation and infrastructure.

## 7. Evidence and Limitations

- **Browser-verified:** Both source views and initial HTML responses; the populated live DOM; literal scarf text without italic elements; both pages after disabling JavaScript and reloading; the inactive CSR mode control; restored JavaScript; the 14-versus-2 request breakdown; and the recorded content-availability measurements.
- **Source-supported:** `public/index.html:39–57` contains the stock placeholder, empty catalogue/template, and module script; `public/panel.js:15–21,39–47` requests `/__mode` and imports C; `public/versions/c.js:14–25,28–37,49–57` fetches, safely renders, and aggregates stock; `server.mjs:67–70,77–107,188–190` escapes names, computes the total, renders HTML, and serves `/ssr`.
- **Measurement limits:** One unthrottled local navigation per page was measured for Question 4. DOM timestamps record content insertion; frame timestamps record pre-paint layout checks, not exact physical display time. They are not an LCP benchmark or a controlled slow-phone experiment. Server stock delays are randomized and browser scheduling varies. The SSR footer timestamp also changes on each request.
- **No fabricated evidence:** No screenshot files were created or claimed. Results came from actual browser/HTTP execution and source inspection, not mocked tests. No student manual execution is claimed. Crawler indexing, other browsers, real slow devices, and exact pixel-presentation latency were not tested.
- **Completion and scope:** All five requested comparisons were executed; no required check is pending. JavaScript was restored, the temporary browser was closed, and the existing server remains in normal mode. Application source, original evidence, verdicts, and Git history were not changed. Phase 6 was not started.

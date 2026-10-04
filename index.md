---
title: Cloud Services — Course Projects
description: Coursework from the Oamk Cloud Services course, autumn 2026
---

# Cloud Services — course projects

Oamk Cloud Services, autumn 2026. **Akseli Polojärvi**

This page is written entirely in Markdown and published with GitHub Pages. It
collects the practical assignments I built during the course, what each one
taught me, and the mistakes worth remembering.

---

## Progress

- [x] Week 1 — Node/Express on Render.com
- [x] Week 2 — written questions on virtualisation and containers
- [x] Week 5 — browser automation with Playwright
- [x] Bundle A — cloud terminology
- [x] Bundle O — GitHub Actions CI
- [x] Bundle E — this page
- [ ] Week 3 — Cloudflare Pages
- [ ] Week 4 — Firebase and Firestore
- [ ] Bundle R — a game built with an AI coding agent

---

## The projects

| Week | Assignment | Platform | Status |
| :--- | :--- | :--- | :---: |
| 1 | Express app reporting its own instance state | Render.com | live |
| 3 | Static site showing the CDN edge that served it | Cloudflare Pages | building |
| 4 | Dynamic site with a database | Firebase | planned |
| 5 | Electricity price scraper | Playwright | done |
| O | Continuous integration | GitHub Actions | passing |

---

## Week 1 — PaaS ephemerality, made visible

The Render quickstart deploys a "hello world" page, which proves nothing: it
looks identical whether it came from a PaaS, a VPS or my own laptop.

Instead I built an app that reports the instance it runs on, with a hit counter
kept deliberately *in process memory*:

```js
// Not stored in a database. A PaaS instance is ephemeral, so this resets
// every time Render spins the free-tier instance down and back up.
let requestCount = 0;
```

Render stops a free instance after ~15 minutes of inactivity. Leaving the page
for an hour and reloading it gives:

| Field | Before | After |
| --- | --- | --- |
| hostname | `…7d6797d778-27nnz` | `…d7cd78dbb-5j98q` |
| uptime | 22s | 11s |
| counter | 5 | 2 |
| commit | `7e7119a` | `7e7119a` |

> The commit is **identical** while the hostname changed. That rules out a
> redeploy — the container was genuinely destroyed and replaced. Statelessness
> observed rather than asserted.

---

## Week 5 — scraping vs. RPA

I wanted electricity spot prices from `porssisahko.net`. First I checked whether
a plain HTTP request would do:

```bash
curl -s https://porssisahko.net/ | grep -c "snt"   # 0
```

Zero. The page ships ~11 kB of HTML and four script tags; every number is drawn
by JavaScript after load. That is the difference between *scraping* a document
and *driving a browser*.

I got the parser wrong twice before getting it right:

1. Read the page as flat text → label came out as `36 Hinta nyt`
   (a live clock digit)
2. Filtered out bare numbers → label came out as `Tilastot Hinta nyt`
   (a navigation link)
3. Walked the **DOM** up from each price until a label appeared → correct

The lesson: ~~proximity on screen~~ **structure in the document**. Both failures
came from treating layout as meaning.

---

## Bundle O — what CI caught on its first run

A workflow running tests across Node 20, 22 and 24. The very first run passed
but attached a warning I had not expected:

```
Node.js 20 is deprecated. The following actions target Node.js 20:
actions/checkout@v4, actions/setup-node@v4
```

I had copied `@v4` from an example without thinking. Nobody would have told me
my build tooling was aging — the automation noticed by itself, for free, on day
one.[^ci]

[^ci]: Upgrading both to `@v5` cleared it. The second run was clean apart from
an informational notice about `ubuntu-latest` migrating to Ubuntu 26.

---

## Things I got wrong

Worth recording, because these taught me more than the parts that worked:

- **`render.yaml` did nothing.** I committed a blueprint declaring the service,
  then configured it by hand in the dashboard — Render only reads that file for
  *Blueprint* deploys, not the manual flow. Config-as-code that isn't applied is
  worse than none, because the file looks authoritative while the real state
  lives elsewhere.
- **I guessed an environment variable name.** `RENDER_REGION` is not something
  Render sets, so the page confidently displayed *"not running on Render"* while
  running on Render.
- **A loose `engines` range.** `">=20.0.0"` let Render install Node 26. My
  production runtime can change without me touching a line of code.

---

## Built with

`Node.js` · `Express` · `Playwright` · `GitHub Actions` · `Render` ·
`Cloudflare Pages` · `Firebase`

---

<sub>Published with GitHub Pages. Source written in Markdown — no HTML.</sub>

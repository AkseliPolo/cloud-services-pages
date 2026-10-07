---
title: Cloud Services - Course Projects
description: Oamk Cloud Services coursework, autumn 2026
---

# Cloud Services course projects

Akseli Polojärvi, Oamk, autumn 2026.

This page is written in Markdown and published with GitHub Pages. It lists the
practical assignments I did on the course and what I learned from each one.

## Assignments

| Week | What I did | Platform |
| --- | --- | --- |
| 1 | Express app that reports its own instance state | Render.com |
| 3 | Static site showing which CDN edge served it | Cloudflare Pages |
| 4 | Dynamic site with a database | Firebase |
| 5 | Electricity price scraper | Playwright |
| O | Tests running automatically on every push | GitHub Actions |
| R | 8-bit game made with an AI coding agent | Godot |

Progress:

- [x] Week 1
- [x] Week 2
- [x] Week 3
- [x] Week 4
- [x] Week 5

## Week 1 - Render.com

I made a small Express app that shows the hostname, uptime and a hit counter.
The counter is kept in memory on purpose:

```js
let requestCount = 0;
```

Render shuts down a free instance after about 15 minutes of no traffic, so if I
leave the page and come back, the counter has reset and the hostname has
changed. That shows what *ephemeral* means in practice.

## Week 3 - Cloudflare Pages

A static page that reads `/cdn-cgi/trace` and shows which Cloudflare datacenter
answered the request. From Oulu I get **HEL**, Helsinki.

> I also learned that `fetch` does not throw on a 404 - it returns a response
> with `ok: false`, so my error handling never ran until I checked for it.

## Week 4 - Firebase

A guestbook where visitors post messages. The messages live in Cloud Firestore
and the page listens for changes instead of fetching once, so a message posted
in one browser appears in every other open browser without a reload.

Cloud Functions need the paid plan, so on the free plan there is no server code
at all. The validation lives in the Firestore security rules, which run on
Google's servers:

```
allow create: if request.resource.data.text.size() <= 280;
```

I checked that this was real by sending a write straight to the Firestore API,
skipping my page completely. It came back `403 PERMISSION_DENIED`, so the rules
really are the server.

## Week 5 - Playwright

A script that collects electricity spot prices. A normal HTTP request does not
work, because the page builds the numbers with JavaScript:

```bash
curl -s https://porssisahko.net/ | grep -c "snt"   # 0
```

I first tried reading the page as plain text, which picked up a clock digit and
then a navigation link into my results. Reading the DOM structure instead fixed
it properly.

## Bundle O - GitHub Actions

Tests run on Node 20, 22 and 24 on every push. The first run already warned me
that the actions I had copied were using a deprecated Node version.

## Bundle R - Godot game

An 8-bit game called *Cold Start*, made with Claude Code in one prompt. The
player is a server instance catching requests, and the instance spins down if
the uptime meter runs out.

---

Source code for the other assignments is in private repositories.

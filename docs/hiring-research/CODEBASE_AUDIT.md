# Codebase audit: swaroopchandrashekar13.github.io (branch feat/global-recruiter-experience)

## Hosting and repo
- GitHub Pages, user site repo, served from `main` root. `.nojekyll` present (files served as-is; any `.md` committed
  to the repo root is publicly reachable at the site URL once on `main`). Only `main` is published; there are no
  preview deployments for other branches.
- Files: `index.html` (≈378 KB), `assets/` (portrait JPG 78 KB, two resume PDFs 43/53 KB, research figure PNG 138 KB),
  `README.md`, `.nojekyll`. No build system, no package.json, no framework, no external scripts or fonts.

## Architecture of index.html
- Single document. `<head>`: meta description, OG/Twitter tags (og:image = portrait), one JSON-LD `Person` block,
  favicon as inline SVG data URI, an inline pre-paint script reading `localStorage['sc-theme']`.
- One `<style>` block (≈51 KB): CSS custom properties on `:root`, dark theme via `html[data-theme=dark]`;
  breakpoints 1450 / 1150 / 900 / 700 / 370 px, `prefers-reduced-motion`, print styles. A "skim layer" set of
  components exists: `.case-summary.glance`, `.kpi-row`, `.points`, `.project-outcome`, `.project-stack`,
  `.metric-strip.four`, `.snapshot`, `.hero-stack`.
- Static header (brand, nav: Work, Research, Notes, About; search button Ctrl/Cmd-K; theme toggle; "Let's talk"
  CTA to contact; mobile menu), static `<main>` holding a copy of the home page (no-JS fallback), static contact band
  and footer (GitHub, LinkedIn, Resume, Site & privacy).
- 26 `<template data-route data-title>` elements: home, work, research, about, resume, contact, notes, lab, privacy,
  404, 13 `work/<slug>` case studies, 3 `notes/<slug>`.
- One `<script>` (≈11 KB): `window.SC_PREVIEW = true` enables hash routing (`#/route?anchor=id`): clicks on
  `a[data-page]` set `location.hash`; `hashchange` swaps `<main>` with the template clone, sets `document.title`,
  nav `aria-current`, re-inits page widgets (work filters with live count, agent-run simulator, system-map buttons,
  reading progress, copy-to-clipboard, toast). Client-side search over an inline `<script type="application/json"
  id="search-data">` index.
- Links are written as real paths (`./work/<slug>/index.html`) with `data-page`; only the hash router makes them work
  (those paths do not exist on disk), so crawlers that do not run JS see only the home copy plus templates' raw HTML.

## Content state
Hero ("I build the platforms that make enterprise AI reliable."), 4 verified metrics, snapshot row (current role,
experience, open to, education), 13 case studies each with a problem/what-I-did/result/stack glance and full depth
sections, resume page with two PDFs (one-page and detailed), about with portrait, contact, research interests page
(three proposed directions; the ongoing Study 1 is being added on this branch), notes, interactive lab.

## Tooling outside the repo (scratchpad)
`tools/split.py` splits index.html into `shell.html` + `pages/<route>.html` and merges back (round-trip exact; merge
regenerates the static `<main>` from the home template). `tools/check_page.py` validates fragments (balanced tags,
links/routes, banned confidential terms, case-study structure). `tools/finalize.py` applies card data. `tools/qa.mjs`
runs Playwright over every route at 1440 and 390 px (errors, overflow, broken links/images, screenshots).

## Gaps relevant to this upgrade
- No region/persona concept; one hero, one ordering, two resumes.
- No availability, time-zone or work-authorization statement anywhere (facts not yet confirmed by the owner).
- No `llms.txt`, no `robots.txt`, no `sitemap.xml`; JSON-LD has only `Person` (no `ProfilePage`, no
  `SoftwareSourceCode`); canonical link absent.
- Deep pages are not separately crawlable URLs (hash routes).
- No scheduling link; contact is email/LinkedIn/GitHub only.

---
title: "VoicePrompter — State of the Project"
filename: "docs/fork/2026-08-05-state-of-project.md"
status: "point-in-time-assessment"
date: "2026-08-05"
generated_by: "claude-fable-5"
audience: ["Product Planner Agents", "Operator", "Dev Agents"]
purpose: >
  Security- and trust-focused snapshot of the aaronse/VoicePrompter fork of
  kosuvorov/VoicePrompter, to support the decision: is it safe to run this on a
  local dev machine, and what would need changing before using or deploying it.
sources:
  - "README.md"
  - "privacy.html"
  - "changelog.md"
  - "docs/google-search-console-setup.md"
  - "package.json / package-lock.json"
  - ".github/workflows/deploy.yml"
  - "cloudflare/gdoc-proxy/worker.js + wrangler.toml"
  - "src/ (main.ts, gdoc.ts, speech.ts, storage.ts, video.ts, platform-promo.ts)"
  - "app/index.html, index.html, web/index.html"
  - "scripts/build-blog.js, build-changelog.js, build-use-cases.js"
  - "update_vite_config.js, test.js, vite.config.ts"
repo_activity:
  last_commit: "2026-08-04"
  quiet_days: 1
  is_active: true            # upstream is very active; the fork itself has zero local commits
invalidated_by: >
  Any local commit by the fork owner (this doc assesses a pristine fork);
  upstream removing or rekeying the tracking stack (app/index.html:65-71);
  upstream deleting the public CORS proxy fallbacks (src/gdoc.ts:54-55);
  a privacy.html rewrite disclosing session recording.
---

> **Superseded in part, 2026-08-06.** This is a dated snapshot of the fork **before** any change was
> made to it, kept as the record of why the fork diverges. Its `invalidated_by` condition below has
> since fired: the fork is no longer pristine. The gaps described in §3 items 1–4 were closed by
> commits `14ac2d7`, `b229e1f`, `0474a91`, `b6903a7`, `0ec588f`. Read
> [README.md](README.md) for the current state; do not treat anything below as present-tense.

# TLDR

VoicePrompter is a Vite/TypeScript voice-controlled teleprompter PWA plus a large SEO/marketing
site, built and actively maintained solely by Konstantin Suvorov (197 commits since 2025-11-22, last
push 2026-08-04). This repo is a **pristine fork** — zero local commits, in sync with upstream. The
supply chain is clean (`real`: registry-pinned lockfile with integrity hashes, no install hooks, no
network/exec in build scripts) and **running `npm install` / `npm run dev` locally is safe**. The
honest gate is trust, not malware: the README's "No External Services / No Data Collection" claim is
**false** (`real`) — the served app loads two trackers plus rrweb-style **session replay** whose
block-list does not cover the script-input textarea, external font CDNs, and a Google-Doc importer
that can fall back to public third-party CORS proxies. All analytics keys, Cloudflare Workers, and
domains belong to the upstream author, so deploying this fork as-is ships his tracking stack.

# 1. What it is intended to do

- A privacy-focused, offline-capable, voice-scrolling teleprompter using only on-device browser
  APIs (README.md:5, README.md:149-154), free web app funneling to paid native iOS/Android/Mac apps.
- Deliberate stances (from README + code): no accounts, no server-side storage of scripts
  (`real` — src/storage.ts uses localStorage only; no upload path exists in src/), GPLv3, PWA-first.
- The privacy stance is the product's core marketing claim — which makes §4's gap load-bearing,
  not cosmetic.

# 2. What is actually built

| Area | Status | Evidence | Label |
|---|---|---|---|
| Teleprompter app core (`src/`, `app/index.html`) | Working, local-only data flow | localStorage at src/storage.ts:6,23,27; getUserMedia; no upload path | real |
| Marketing site (60+ pages: blog, /mac /ios /ipad /android /web, use-case generator) | Built, heavily SEO-optimized | scripts/build-*.js generate pages at build time, pure local file I/O | real |
| Google Doc import | Working via author's Cloudflare Worker; 2 public-proxy fallbacks | src/gdoc.ts:52-56; cloudflare/gdoc-proxy/worker.js:19-27 (Origin allowlist, rate limit) | real |
| PWA/service worker | Default Workbox precache, no external origins, no runtimeCaching | vite.config.ts:31-36 | real |
| CI deploy | GitHub Pages via peaceiris action on push to main **or staging** | .github/workflows/deploy.yml:3-8,30-34 | real |
| Analytics | Umami (self-hosted, railway.app) + session replay + Ansvisor AI-referral tracker | app/index.html:65-71; 59 pages carry Ansvisor | real |
| Native apps (iOS/Android/Mac) | **Not in this repo** — marketing pages only, no native source, no binaries | android/, ios/, mac/ each contain a single index.html | real |
| Supply chain | Clean: 568/568 lockfile entries registry.npmjs.org with integrity; no pre/postinstall; no eval/exec/base64 blobs anywhere | package-lock.json; package.json:6-12 | real |
| Test harness | Vestigial: test.js imports puppeteer (not a dependency); test-history.js is 41 bytes | test.js:1; package.json | seam |

# 3. Ambition vs. implemented reality — the gaps

1. **"No Data Collection" vs. three trackers on the app page.** README.md:5,151-152 promise no
   external services. Reality: Umami analytics (app/index.html:65), Ansvisor third-party tracker
   (app/index.html:68 — remote minified script, unauditable from source), and session replay at 30%
   sampling (app/index.html:69-71). This is **drift**, not deferral — the README was last
   substantively touched 2026-06-29; the tracking stack has grown since 2026-05-09.
2. **Session replay block-list misses the script input.** `data-block-selector` covers
   `#scriptContent, #historyList` but not `#inputScript` (the paste textarea, app/index.html:227)
   or `#googleDocUrlInput` (app/index.html:160). Whether `data-mask-level="moderate"` masks
   textarea values depends on the remote recorder.js — `claimed`/unverifiable from this repo.
3. **privacy.html never mentions session recording** (privacy.html:268 claims "anonymized… no
   PII"). A live policy/practice mismatch on the production site.
4. **"Completely offline" vs. reality**: Google Fonts + cdnfonts.com load inside /app/
   (app/index.html:56-62, no SRI); Chrome's Web Speech API streams audio to Google (browser
   property, src/speech.ts:25-35); Doc import can route document bodies through
   api.allorigins.win / corsproxy.io (src/gdoc.ts:54-55 — code comment itself calls them legacy
   and unreliable).
5. **Fork-specific**: every piece of infrastructure (analytics endpoints, both Cloudflare Workers,
   the Ansvisor key, voiceprompter.app domain) is the upstream author's. The Worker's Origin
   allowlist (worker.js:19-24) does not include any fork domain — deploy this fork elsewhere and
   Doc import silently falls through to the public proxies.

# 4. Age and activity

- Upstream is in a hot streak: 30 commits since 2026-07-05, all SEO/marketing-focused. App code
  (`src/`) last materially touched 2026-06-29–2026-07-24.
- Oldest unrevisited load-bearing thing: **deploy.yml (2025-11-22, 256 days)** — still deploys the
  `staging` branch with `contents: write` to the same gh-pages target as main.
- Fossils: update_vite_config.js (2026-02-27) hard-codes `/Users/konstantin.suvorov/…` and would
  crash (CommonJS require in an ESM package); test.js (2026-05-10) imports a dependency that
  doesn't exist.
- Expired assumption: src/gdoc.ts's public-proxy fallbacks were observed dead in July 2026 by the
  code's own comment (src/gdoc.ts:50-51) yet remain in the request path.
- Commit-hygiene signal: commit 0a61978 ("Improve SEO content and site navigation", 2026-08-04)
  silently swapped the Ansvisor tracker host `api.ansvisor.com` → `ansvisor-api.konsu.dev` and
  changed the site key across 59 pages. konsu.dev is the author's own domain (about.html:411), so
  this reads as self-hosting, not a hijack — but tracker changes hidden under SEO commit messages
  is exactly the pattern a fork reviewer must watch for here.

# 5. What is next

The repo's own docs don't state a roadmap beyond SEO growth (docs/google-search-console-setup.md is
an operator checklist). For **this fork**, the next step is a decision, not dev work: strip or
replace the upstream tracking/infra endpoints before any local use beyond experimentation, and
before any deployment. That is ~10 lines across app/index.html and src/gdoc.ts.

# 6. Cross-project relationships

None found in-repo. The native iOS/Android/Mac apps live elsewhere (closed or private); this repo
only markets them. The two Cloudflare Workers' deployed copies live in the upstream author's
dashboard — worker.js:15-16 explicitly warns the repo copy and deployed copy must be kept in sync
manually, so the repo copy is `claimed` as the deployed behavior, not `real`.

# 7. Reading list + authority chains

1. This document.
2. [work-scout.md](work-scout.md) — the actionable ledger.
3. src/gdoc.ts + cloudflare/gdoc-proxy/worker.js — the only user-content network path.
4. app/index.html:56-71 — the entire tracking stack, in one block.
5. package.json / package-lock.json — supply-chain ground truth.

Authority: **code beats README everywhere** (README's privacy section is marketing and provably
stale). For proxy behavior, the deployed Worker beats the repo copy (unverifiable; treat repo copy
as intent). For deploy behavior, deploy.yml is authoritative.

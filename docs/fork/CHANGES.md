# Fork changelog

Every change this fork makes to upstream `kosuvorov/VoicePrompter`, newest first. This is the
authoritative log — [README.md](README.md) explains *why* the fork exists, this file records *what
it changed*. No pull requests are planned; entries are written so they can be cherry-picked.

Product-facing release notes live in the repo root (`changelog.md`) and belong to upstream. Do not
add fork entries there.

---

## 2026-08-06

### Made the GitHub Pages deploy manual-only

`.github/workflows/deploy.yml` now triggers on `workflow_dispatch` instead of on push to `main`.

Actions are enabled on this fork and the workflow was registered and active, so the next push to
`main` would have built the site and pushed `dist/` to a public `gh-pages` branch — standing up a
duplicate of upstream's marketing site, carrying their canonical URLs, App Store campaign links
and contact address, under a different account. This fork is run locally and has nothing to
publish.

The secondary reason is supply chain: the workflow uses `peaceiris/actions-gh-pages@v3`, a
floating tag rather than a pinned SHA, and the repository allows all actions. That is an
acceptable risk for a workflow you rely on and a pointless one for a workflow you never use.

Chosen over deleting the file, so upstream's deploy logic stays available for reference, and over
disabling it through the GitHub API, which is repository state rather than code — invisible in a
clone and silently undone by anyone who re-enables Actions. The restore instructions are in a
comment at the top of the file.

**Not for cherry-picking.** Upstream needs its automatic deploy; this is a fork-only change.

Related: `0474a91` earlier removed the `staging` branch from the same trigger.

### Removed three dead root-level scripts

`test.js`, `test-history.js` and `update_vite_config.js` were one-off debugging artifacts from
February–May 2026. None was referenced by an npm script, and each was broken on execution:
`test.js` imported `puppeteer`, which is in neither dependency list; `update_vite_config.js` used
CommonJS `require` in a `"type": "module"` package and targeted a hard-coded
`/Users/konstantin.suvorov/Teleprompter/vite.config.ts`; `test-history.js` was a single
`console.log`.

Deleted rather than repaired. A file-writing script in the root of a public repository is
something every security reviewer must stop and read, and a test entry point that fails to resolve
its own import invites someone to `npm i puppeteer` — pulling a Chromium download into the tree —
in order to "fix" a file nothing uses.

**Not worth cherry-picking as-is.** Upstream may still want these files, or may prefer to make
`test.js` work by adding the dependency. This is a tidiness call specific to a fork that gets
audited.

### Fixed: blog generator duplicated every post's `<h1>` on Windows checkouts

`scripts/build-blog.js` strips a leading markdown `# Title` from each post body because the article
template already emits `<h1>{{TITLE}}</h1>`. On a Windows checkout with `core.autocrlf=true` the
markdown sources arrive with CRLF line endings, and the strip pattern `/^\s*# .*\n+/` cannot match:
in JavaScript regex `.` does not match `\r`, so `.*` stops short of the line ending and the required
`\n+` never matches. With no `m` flag there is no second anchor position, so the replace silently
no-ops.

Effect: 26 of 40 posts shipped two `<h1>` elements — reintroducing the exact defect upstream fixed
in `58ad897` ("fix duplicate H1s") — and because the generated HTML is committed, a rebuild looked
like an ordinary diff rather than a regression.

Fix: normalise CRLF to LF at the file read, rather than patching the one regex. The same trap sits
in every other line-oriented pattern in the file (the trailing-whitespace trim among them), and
normalising at the boundary makes generated HTML byte-identical across platforms. It is a no-op on
macOS and Linux, so it cannot regress upstream's own builds.

**Worth cherry-picking upstream.** This is a cross-platform build-correctness bug that silently
degrades the SEO of 26 pages for any Windows contributor. It is unrelated to anything opinionated
about this fork.

### Privacy hardening

Removed the upstream telemetry and third-party request paths from the app. The teleprompter core
was already local-only — no upload path exists in `src/`, scripts live in `localStorage` — so these
changes affect the layer around it, not the product.

| Commit | Change |
|---|---|
| `14ac2d7` | Removed Umami analytics, the Ansvisor tracker, and 30%-sampled session replay from `app/index.html` |
| `b229e1f` | Removed the public CORS-proxy fallbacks from Google Doc import; it now uses only the origin-allowlisted Cloudflare Worker and fails closed |
| `0474a91` | Stopped the `staging` branch from auto-publishing to the same GitHub Pages target as `main` |
| `b6903a7` | Rewrote the privacy claims in `README.md` and `privacy.html` to match what the code does |
| `0ec588f` | Self-hosted DM Sans and OpenDyslexic under `public/vendor/` with their OFL/SIL licences |

Verified: `npm run build` exits 0, `dist/app/index.html` contains no external references, and the
built app bundle contacts exactly one host — the Google Doc proxy Worker, and only on import.

`0ec588f` is the other change worth cherry-picking. It removes two third-party CDN dependencies so
the app loads with no external request at all, and it keeps the OpenDyslexic accessibility option
working — naive deletion of the CDN link would have broken it for dyslexic users.

Reasoning and evidence: [work-scout.md](work-scout.md) · [2026-08-05-state-of-project.md](2026-08-05-state-of-project.md)

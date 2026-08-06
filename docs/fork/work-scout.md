# VoicePrompter — Work Scout Ledger (2026-08-05)

State doc: [2026-08-05-state-of-project.md](2026-08-05-state-of-project.md)
Machine-readable: [work-scout.json](work-scout.json)

> **Status, 2026-08-06:** W1–W4 are **shipped** (commits `14ac2d7`, `b229e1f`, `0474a91`, `b6903a7`,
> plus `0ec588f` self-hosting the fonts). W5 is open; W6 resolved to *no work* under decision D1=b.
> See [README.md](README.md) for the current state — this ledger is kept as the reasoning record.

Pristine fork of kosuvorov/VoicePrompter (zero local commits, in sync 2026-08-04). Supply chain
verified clean; every candidate below is about trust/privacy posture, not malware.

| # | Title | Shape | Age | Route | Cost of delay |
|---|---|---|---|---|---|
| W1 | Strip upstream tracking stack from /app/ | drift | 1d (surface changed yesterday) | just-do-it | Local sessions leak usage + sampled DOM recordings to upstream |
| W2 | Remove public CORS-proxy fallbacks from Doc import | aging-assumption | 21d | just-do-it | Doc content transits third parties on Worker failure or non-allowlisted deploy |
| W3 | README/privacy.html contradict shipped code | doc-rot | 37d | just-do-it (after W1) | Nothing locally; reputational/legal if published as-is |
| W4 | `staging` branch auto-deploys to production gh-pages | silent-failure | **256d** | just-do-it | Zero until fork Actions enabled; then one push publishes experiments |
| W5 | Delete fossils: update_vite_config.js, test.js, test-history.js | fossil | **159d** | just-do-it — **shipped** | Nothing much; auditor attention on every sync |
| W6 | Fork runs entirely on upstream author's infrastructure | untested-assumption | structural | **split** → D1 decision + W6a batch | Zero under D1=a/b; a launch blocker under D1=c |

Implementation prompt for W1–W4 (executed 2026-08-06): [privacy-hardening.md](privacy-hardening.md)

Quota check: 2/6 candidates >90 days — **not met, honestly**: upstream rewrote most load-bearing
surface within the last 90 days; both genuinely old items that still matter are listed.

---

## W1 — Strip upstream tracking stack from /app/ (drift, just-do-it)

```text
Problem: The app page loads Umami analytics, the unauditable Ansvisor remote script, and rrweb
         session replay whose block-list omits the script-input textarea, while README and
         privacy.html claim no data collection.
Cause:   Growth instrumentation accreted after the privacy docs were written; the fork inherits
         the author's keys and endpoints wholesale.
Fix:     Delete app/index.html:65-71 (optionally 56-62, external fonts); app works without them.
Impact:  Fork delivers the privacy the README promises; nothing reaches upstream analytics.
Risk:    Merge conflicts on future upstream pulls touching the app/index.html head.
Goal:    Loading /app/ makes zero requests to reactive-analytics.up.railway.app or
         ansvisor-api.konsu.dev (verifiable in DevTools).
```

Evidence: [app/index.html:56-71](../../app/index.html#L56-L71) (trackers + replay),
[app/index.html:227](../../app/index.html#L227) (`#inputScript` not in block-selector),
[README.md:151-152](../../README.md#L151-L152), [privacy.html:268](../../privacy.html#L268).
Note: commit 0a61978 (2026-08-04, "Improve SEO content and site navigation") silently swapped the
Ansvisor host and site key across 59 pages — this surface changes under unrelated commit messages.

**Steelman against:** localhost-only code reading fires trackers against upstream's own analytics
and harms nobody but your data trail; deletion costs a permanent merge seam against a fast upstream.

## W2 — Remove public CORS-proxy fallbacks (aging-assumption, just-do-it)

```text
Problem: If the author's Cloudflare Worker fails, users' Google Doc contents are fetched through
         api.allorigins.win / corsproxy.io — third parties see doc ID and full body.
Cause:   Proxies predate the self-hosted Worker (2026-07-15); the code's own comment says all
         three were down in July 2026 and may never recover.
Fix:     Delete src/gdoc.ts:54-55.
Impact:  Document content can no longer transit an unaccountable third party.
Risk:    Doc import fails loudly when the Worker is down instead of silently degrading.
Goal:    No request path sends user document identifiers or content outside the deployer's
         control plus docs.google.com.
```

Evidence: [src/gdoc.ts:50-56](../../src/gdoc.ts#L50-L56),
[cloudflare/gdoc-proxy/worker.js:19-27](../../cloudflare/gdoc-proxy/worker.js#L19-L27).

**Steelman against:** docs must already be public-by-link to import at all, so marginal exposure is
the doc ID (a capability URL) plus proxy-operator visibility — real but bounded.

## W3 — README/privacy.html contradict shipped code (doc-rot, just-do-it after W1)

```text
Problem: README says "No External Services / No Data Collection" and privacy.html omits session
         recording, while the app ships three telemetry loaders and external font CDNs.
Cause:   Privacy copy (last touched 2026-06-29) was never revisited as instrumentation accreted
         from 2026-05-09 onward.
Fix:     Do W1 first, then rewrite the privacy claims to match actual behavior.
Goal:    Every privacy claim is verifiable against the network behavior of a built /app/ page.
```

Evidence: [README.md:5](../../README.md#L5), [README.md:149-154](../../README.md#L149-L154),
[privacy.html:268](../../privacy.html#L268), [app/index.html:65-71](../../app/index.html#L65-L71).

**Steelman against:** once W1 lands the claims become mostly true; arguably fold into W1.

## W4 — staging branch auto-deploys to production gh-pages (silent-failure, just-do-it)

```text
Problem: deploy.yml triggers on push to main OR staging, both publishing ./dist to the same
         gh-pages target — pushing any branch named "staging" silently replaces the live site.
Cause:   Staging deploys enabled on repo day one (2025-11-22), never scoped to a separate target.
Fix:     Drop "staging" from the trigger in the fork, or disable the workflow until deployment
         is intended.
Risk:    None; the fork has no staging flow.
Goal:    Only main can publish, or the workflow is disabled.
```

Evidence: [.github/workflows/deploy.yml:3-8](../../.github/workflows/deploy.yml#L3-L8),
[deploy.yml:30-34](../../.github/workflows/deploy.yml#L30-L34). Oldest live config in the repo
(256 days untouched).

**Steelman against:** GitHub disables workflows on forks until manually enabled; the trap is armed
only if Actions are turned on.

## W5 — Delete fossils (fossil, just-do-it) — *clarified per BATCH-DEV PCFIRG rules*

```text
Problem: Both root-level scripts fail on execution — `node test.js` exits non-zero with
         ERR_MODULE_NOT_FOUND (puppeteer is in neither dependency list) and
         `node update_vite_config.js` crashes on a hard-coded /Users/konstantin.suvorov/ path
         — so every audit of this fork must prove two file-writing scripts inert first.
Cause:   One-off dev artifacts committed Feb-May 2026; no npm script invokes them, so nothing
         ever surfaced the breakage.
Fix:     Delete update_vite_config.js, test.js, test-history.js.
Goal:    No script outside scripts/ and the build config remains, and `npm run build` exits 0.
```

Hard exit criterion: `npm run build` exits 0 **and** `Test-Path` is false for all three files.

**Shipped 2026-08-06** — all three deleted; see [CHANGES.md](CHANGES.md). The evidence below cites
files that no longer exist, so the paths are given unlinked: `update_vite_config.js:2` and `:10`
(the hard-coded macOS path and the `fs.writeFileSync` to it), `test.js:1` (the unresolvable
`puppeteer` import).

**What changed and why:** the original Problem described *file contents* ("repo root carries a
script…"), which is a state, not a defect — it drifted toward restating the Fix in the negative.
Restated as the observable failure (two non-zero exits). Impact and Risk are dropped under the
contract's merge-away rule: neither changed the decision.

**Verdict on justification:** justified but marginal — real friction, no user-visible defect.
Batch overhead (fresh session, plan, handoff) exceeds a three-file deletion, so ride it along as a
fifth commit on the W1–W4 branch rather than spawning a session for it.

**Steelman against:** they are inert; deleting buys tidiness, not safety, and adds merge friction.

## W6 — Fork runs on upstream's infrastructure — *split; the original header failed PCFIRG*

**Why it failed the contract:** the Problem was conditional ("deployed anywhere else, it *would*
lose Doc import") rather than an observable defect today, and the Goal was "write down a decision"
— a document, not a checkable state, so no hard exit criterion could exist. Both are named
prohibitions in the batch-planning contract this fork's notes follow (PCFIRG headers). It splits
into a decision and one conditional batch.

### D1 — Owner decision (not Dev work)

**What is this fork for?**

- **(a) Read-only study of the code** — no work at all; W1–W4 optional, W6 evaporates.
- **(b) Private tool on a local dev machine** *(the stated situation)* — fully satisfied by W1+W2.
  W6 needs **nothing further**: `localhost:5173` and `localhost:4173` are already in the Worker's
  allowlist ([worker.js:19-24](../../cloudflare/gdoc-proxy/worker.js#L19-L24)), so Doc import keeps
  working from `npm run dev` after W2.
- **(c) Independent deployment under another domain** — unlocks W6a.

Under (a) and (b) there is no W6 work. Only (c) produces a batch.

### W6a — Self-host the Doc proxy (conditional on D1 = c)

```text
Problem: From any origin other than voiceprompter.app or localhost the gdoc Worker returns 403,
         and once W2 removes the fallbacks Doc import fails with a generic "try again later"
         message that names no cause.
Cause:   Endpoint hosts and the Worker's ALLOWED_ORIGINS are hard-coded literals belonging to
         upstream; nothing in the fork parameterizes them.
Fix:     Deploy the fork's own copy of cloudflare/gdoc-proxy with the fork's origins in
         ALLOWED_ORIGINS, and point src/gdoc.ts at it.
Impact:  The fork owns every endpoint it depends on.
Risk:    Needs a Cloudflare account and a rate-limiter binding, and the deployed Worker inherits
         the manual repo-vs-dashboard sync burden the file itself warns about.
Goal:    Importing a public Google Doc from the fork's deployed origin returns its text.
```

Hard exit criterion: `curl -H 'Origin: <fork-origin>' <fork-worker>/?id=<known-public-doc>`
returns 200 with `text/plain`.

Route: **batch** (re-routed from just-do-it) — under D1=c this is gated implementation work with a
curl-shaped exit code. It was never a study; nothing here is unknown.

Evidence: [src/gdoc.ts:53](../../src/gdoc.ts#L53),
[worker.js:15-16](../../cloudflare/gdoc-proxy/worker.js#L15-L16) (sync warning),
[worker.js:39-41](../../cloudflare/gdoc-proxy/worker.js#L39-L41) (the 403 path),
[index.html:1752](../../index.html#L1752).

**Steelman against:** if the fork exists only to read code or send upstream PRs, decoupling is
wasted work and permanent merge friction — and self-hosting buys a sync obligation the repo
already admits it handles by hand.

---

## Considered and dropped

- **Malicious-code findings** — none: no eval/new Function/base64 blobs/install hooks; 568/568
  lockfile entries registry-pinned with integrity; build scripts are pure local file I/O.
- **Worker repo-copy vs deployed-copy drift** — real seam (worker.js:15-16) but unactionable from
  the fork; the deployment lives in upstream's dashboard.
- **YouTube iframes not privacy-enhanced** — marketing pages only; below the bar.
- **Chrome Web Speech API streams audio to Google** — browser property, not repo code.
- **Missing SRI on font CDNs** — folded into W1's optional scope.

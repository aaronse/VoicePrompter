# Privacy hardening — the implementation prompt for W1–W4

> **Executed 2026-08-06.** This ran and its work is landed (commits `14ac2d7`, `b229e1f`,
> `0474a91`, `b6903a7`; the font self-hosting in `0ec588f` was a follow-up beyond this prompt's
> scope). **Do not re-run it against this fork** — the edits it describes are already applied and
> its line numbers refer to the pre-change files. It is kept for two reasons: it documents exactly
> what was changed and why, and it is a reusable recipe for anyone forking this project (or the
> upstream one) who wants the same result.
>
> Reusing it elsewhere: re-derive every line number first. Everything else — the anchors, the
> reasoning, the scope boundaries, the gate — transfers as-is.

Copy everything below the line into a fresh agent session. It is self-contained: every file, line
range, anchor string and replacement is primed, so no agent in this run needs to search or read a
large file to find its edit.

---

You are the **orchestrator** for a four-item privacy-hardening change set in `e:\git\VoicePrompter`,
a pristine fork of `kosuvorov/VoicePrompter` (zero local commits, synced 2026-08-04). You will
implement W1, W2, W3, W4 **in that order**, delegating each to **one fresh subagent**, then run a
single gate at the end.

## Why the work exists

The fork inherits the upstream author's telemetry and infrastructure wholesale. The app's own
README claims "No External Services / No Data Collection" while `/app/` loads two trackers plus
session replay. Full assessment: `docs/fork/2026-08-05-state-of-project.md`.
Ledger: `docs/fork/work-scout.md`. **Do not read either unless you get blocked** —
everything you need is in this prompt.

## Hard rules

1. **Work on a branch.** First action: `git checkout -b fork/privacy-hardening`. `main` stays
   merge-clean against upstream. One commit per work item, message = the item title.
2. **Never read `app/index.html`, `index.html`, or `web/index.html` whole** — they are 58 KB, 92 KB
   and 107 KB. Read only the line ranges named below. This rule is the main cost control in the run.
3. **You (the orchestrator) read no source files at all.** You spawn, collect receipts, gate, report.
4. **One subagent per item, spawned only after the previous receipt is in.** W3 depends on W1's
   result; the others are ordered for reviewability.
5. **Scope is exactly the four items.** No refactors, no dependency changes, no reformatting, no
   touching `src/main.ts` or `src/video.ts`. If an item needs a decision not answered here, the
   subagent stops and reports `BLOCKED` with the one question — it does not improvise.
6. **Run the gate once, at the end.** Not per item.
7. If `node_modules/` is absent, run `npm ci` before the gate (not before the edits).

## Subagent contract (applies to all four)

Give each subagent only its own item block below, plus these instructions:

> You are making one bounded edit. Read only the line ranges named. Use exact-string Edit calls
> against the anchors given — do not search for them. Do not run the build. Do not commit. Do not
> read any other file. When done, reply with **at most 8 lines** in this form:
> ```
> ITEM: <id>
> STATUS: DONE | BLOCKED
> EDITS: <file:line-range> — <what changed>, one line each
> VERIFY: <the grep/Select-String you ran and its result>
> NOTE: <only if something differed from the prompt's expectation>
> ```
> Nothing else. No summary, no explanation, no diff paste.

---

## W1 — Strip the tracking stack from the app page

**File:** `app/index.html`. **Read range:** lines 54–75 only.

Delete lines **64–71 inclusive** — the two HTML comments and all three `<script>` tags:

```html
    <!-- Umami Analytics -->
    <script defer src="https://reactive-analytics.up.railway.app/script.js"
        data-website-id="eee38a2c-9b72-4937-9ee4-37827bae546c"></script>
    <!-- Ansvisor AI-traffic tracking -->
    <script src="https://ansvisor-api.konsu.dev/t.js" data-t="52654f870814a45e679ea63d95079b41" defer></script>
    <script defer src="https://reactive-analytics.up.railway.app/recorder.js"
        data-website-id="eee38a2c-9b72-4937-9ee4-37827bae546c" data-sample-rate="0.3" data-mask-level="moderate"
        data-max-duration="300000" data-block-selector="#scriptContent, #historyList"></script>
```

Leave the blank line structure tidy: line 62 (`fonts.cdnfonts.com`) should be followed by `</head>`.

**Primed facts — do not re-derive:**
- Every analytics call site in `src/` is optional-chained (`(window as any).umami?.track(...)`, 22
  call sites across `src/main.ts` and `src/video.ts`). Removing the script tags **cannot** throw.
  **Do not edit `src/`.**
- Lines **56–62 (Google Fonts + `fonts.cdnfonts.com`) are deliberately OUT of scope.** OpenDyslexic
  is a live in-app accessibility feature (`src/render.ts`, `src/elements.ts`); deleting the CDN link
  breaks it. Self-hosting those fonts is a separate follow-up, not this item.
- Only `app/index.html` is in scope. The ~58 marketing pages keep their tags; they are not the app
  and rewriting them all is a large diff for no privacy gain to a local user.

**Done when:** `Select-String -Path app/index.html -Pattern 'reactive-analytics|ansvisor'` returns
nothing.

---

## W2 — Remove the public CORS-proxy fallbacks from Google Doc import

**File:** `src/gdoc.ts`. **Read range:** lines 41–96.

Replace the `proxies` array (lines 52–56) so only the origin-allowlisted Cloudflare Worker remains:

```ts
    const proxies = [
        `https://gdoc-proxy.kosuvorov.workers.dev/?id=${docId}`
    ];
```

Delete the now-wrong comment on lines 50–51 about "public CORS proxies below are legacy fallbacks"
and replace it with one line stating that Doc import goes only through the origin-allowlisted Worker
(`cloudflare/gdoc-proxy/worker.js`) and fails closed.

`exportUrl` (line 47) becomes unused once the fallbacks are gone — delete that statement too, or
`tsc` will not necessarily complain but the dead code is misleading. Keep `extractDocId`,
`fetchWithTimeout`, the loop, and all error handling exactly as they are.

**Primed fact:** the Worker's Origin allowlist (`cloudflare/gdoc-proxy/worker.js:19-24`) covers
`voiceprompter.app`, `www.voiceprompter.app`, `localhost:5173`, `localhost:4173` — so Doc import
works from `npm run dev` on this fork but 403s from any other deployed origin. That is the intended
fail-closed behavior after this change; do not "fix" it by re-adding a proxy.

**Done when:** `Select-String -Path src/gdoc.ts -Pattern 'allorigins|corsproxy'` returns nothing and
the file still exports `extractDocId` and `fetchGoogleDocText`.

---

## W3 — Make the privacy claims true (run after W1 lands)

**Files:** `README.md` (read lines 1–10 and 145–156 only) and `privacy.html` (read lines 258–280 only).

In `README.md`:
- Line 5 — the phrase "no external APIs, completely private" is now true for the app's own
  telemetry, but not for fonts or Chrome's speech backend. Rewrite the sentence to say processing is
  on-device and the app sends no telemetry, and stop claiming zero external requests.
- Lines 149–154 (`## 🔐 Privacy & Security`) — replace the bullet "**No External Services** —
  Everything runs locally in your browser" with an honest pair of bullets: no analytics or session
  recording in this fork; web fonts still load from Google Fonts and `fonts.cdnfonts.com`; on Chrome
  the Web Speech API sends audio to Google's servers (a browser property, not the app's doing);
  Google Doc import contacts a Cloudflare Worker with the document ID only.
- Add one short "Fork notes" section near the top stating this is a fork of
  `kosuvorov/VoicePrompter` with the upstream telemetry removed.

In `privacy.html`:
- Line 268 (`<h2>Website Analytics</h2>` paragraph) — the marketing pages still carry Umami and the
  Ansvisor tracker, and until W1 the app also ran 30 %-sampled session replay. Rewrite the paragraph
  to name what actually runs and where, and state plainly that the teleprompter app itself carries
  no analytics or session recording.
- Line 271 (Voice Processing) — add one sentence that on Chromium browsers the Web Speech API
  transmits audio to the browser vendor's servers.

**Constraint:** prose edits only. Do not restructure either document, do not touch any other
`<h2>` section, and do not add a changelog entry.

**Done when:** neither file contains the strings "No External Services" or "No Data Collection", and
`privacy.html` contains a sentence naming session recording's removal from the app.

---

## W4 — Stop the staging branch from auto-publishing

**File:** `.github/workflows/deploy.yml` (25 lines — read it whole; it is the one file small enough).

Remove `staging` from the push trigger so only `main` can publish:

```yaml
on:
  push:
    branches:
      - main
```

Leave the rest of the workflow untouched.

**Done when:** `Select-String -Path .github/workflows/deploy.yml -Pattern 'staging'` returns nothing.

---

## Gate — run once, after all four receipts are DONE

```powershell
if (-not (Test-Path node_modules)) { npm ci; if ($LASTEXITCODE -ne 0) { 'NPM CI FAILED'; exit 1 } }
npm run build; if ($LASTEXITCODE -ne 0) { 'BUILD FAILED'; exit 1 }
$leaks = Select-String -Path 'app/index.html','src/gdoc.ts','dist/app/index.html' `
    -Pattern 'reactive-analytics|ansvisor|allorigins|corsproxy' -ErrorAction SilentlyContinue
$stage = Select-String -Path '.github/workflows/deploy.yml' -Pattern 'staging' -ErrorAction SilentlyContinue
$claim = Select-String -Path 'README.md' -Pattern 'No External Services|No Data Collection' -ErrorAction SilentlyContinue
if ($leaks -or $stage -or $claim) { $leaks; $stage; $claim; 'GATE FAILED'; exit 1 }
'GATE PASSED'; exit 0
```

`dist/app/index.html` is the built app page — checking it, not just the source, is what proves the
tracker did not survive through the build or the service-worker precache.

**On gate failure:** report the failing check and stop. Do not retry more than once, and never
"fix" a failure by widening scope beyond the four items.

## Report format (your final message, ≤20 lines)

```
BRANCH: fork/privacy-hardening
W1 <status> — <one line>
W2 <status> — <one line>
W3 <status> — <one line>
W4 <status> — <one line>
GATE: PASSED | FAILED — <the failing check, if any>
COMMITS: <sha short> x4
RESIDUAL: <anything a human must still decide>
```

# Agent instructions

Read this before changing anything. It is the single agent contract for this repo — Claude Code,
Codex/ChatGPT, and anything else. `CLAUDE.md` only imports this file; do not duplicate guidance
into it, or the two will drift.

## What this repo is

A **public fork** of [`kosuvorov/VoicePrompter`](https://github.com/kosuvorov/VoicePrompter), a
voice-controlled teleprompter PWA plus a large SEO marketing site. Upstream is actively maintained
and this fork tracks it. No pull requests are planned — changes are documented so upstream or
anyone else can cherry-pick them.

## The `docs/fork/` convention

**All fork-specific notes live in `docs/fork/`, and nowhere else.** That folder is not product
documentation and contains nothing from upstream. It exists so this fork's divergence is legible
to a reader who did not write it.

| Path | Holds |
|---|---|
| `docs/fork/README.md` | Why the fork diverges; index to the rest |
| `docs/fork/CHANGES.md` | **Authoritative log of every change this fork makes.** Newest first, dated |
| `docs/fork/work-scout.md` / `.json` | Candidate work ledger, with a counter-argument per item |
| `docs/fork/2026-08-05-state-of-project.md` | Dated pre-change assessment. History, not current state |
| `docs/fork/privacy-hardening.md` | The prompt that implemented the privacy work. **Already executed — do not re-run it** |

Rules:

- **Any change this fork makes to upstream code gets a dated entry in `docs/fork/CHANGES.md`.**
  Say what changed, why, and whether it is worth cherry-picking upstream.
- `changelog.md` and `changelog.html` in the repo root are **upstream's product changelog**. Never
  add fork entries there.
- Do not create fork notes elsewhere — no `docs/state/`, no `research/`, no root-level scratch
  files. One home per fact.
- Everything in `docs/fork/` is public. Write accordingly.

## Invariants — do not break these

1. **No telemetry in the app.** `app/index.html` must load no analytics, session recorder, or
   third-party script. This is the reason the fork exists. The check: the built
   `dist/app/index.html` has no external references, and the app bundle contacts exactly one host
   (the Google Doc proxy Worker, only on import).
2. **No new external request paths in `src/`.** Fonts are self-hosted under `public/vendor/`. Do
   not reintroduce a CDN link, and do not add a fallback proxy to `src/gdoc.ts` — it fails closed
   by design.
3. **Privacy claims in `README.md` and `privacy.html` must match the code.** If you change what
   the app requests, update both in the same commit.
4. **Do not commit unless asked.** Show the diff and wait.

## Build gotchas — read before running the build

- **Generated HTML is committed.** `blog/*.html`, `mac/*/index.html` and `changelog.html` are build
  outputs *and* tracked files. Running `npm run build` rewrites them, so it dirties the working
  tree as a side effect. Check `git status` before and after, and do not sweep those files into an
  unrelated commit.
- `blog/index.html` carries a `dateModified` stamp of the build date, so it always shows a diff
  after a build. That one is expected; discard it.
- **Line endings.** Sources are checked out CRLF on Windows. Any line-oriented regex in
  `scripts/build-*.js` must tolerate `\r` — in JavaScript regex `.` does **not** match `\r`, so a
  pattern like `/^# .*\n/` silently fails to match and no-ops without an error. `build-blog.js`
  normalises to LF at the file read; do the same in any new generator.
- Sources of truth: `blog/*.md` generate `blog/*.html`; `scripts/use-cases.json` generates
  `mac/*/`. Never hand-edit a generated `.html` — the next build reverts it.

## Verifying a change

```powershell
npm run build   # must exit 0
Select-String -Path 'app/index.html','src/gdoc.ts','dist/app/index.html' `
    -Pattern 'reactive-analytics|ansvisor|allorigins|corsproxy'   # must return nothing
git status --short   # expect only your intended files, plus possibly blog/index.html
```

## Working style

- Repository evidence outranks any document, including this one. If they disagree, the code wins
  and the disagreement is worth reporting.
- Read file ranges, not whole files. `index.html` is 92 KB, `web/index.html` is 107 KB.
- State what you verified versus what you inferred. Do not report work as done without evidence.

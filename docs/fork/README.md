# Fork notes

**These are not product documentation.** Nothing in this folder is part of the VoicePrompter app,
and none of it comes from the upstream project. It is one fork owner's working notes: an assessment
of the codebase as it stood on 2026-08-05, the reasoning behind the changes made to this fork, and
the prompt used to make them.

It lives in the repo — rather than in a private notebook — so that the changes on this branch are
legible to anyone who reads them: the upstream author, other forks, and coding agents working in
this tree. **Take anything here that is useful to you.** No pull requests are planned; the changes
are documented rather than proposed, so they can be cherry-picked, adapted, or ignored.

## Why this fork diverges

The upstream project is a genuinely good teleprompter with an unusual property: the core app really
does keep your script on your device. There is no upload path anywhere in `src/`, scripts live in
`localStorage`, and speech recognition is the browser's own. That is worth preserving, and it is
why this fork exists rather than a rewrite.

What this fork changes is the layer around that core. As of 2026-08-05 the app page loaded two
analytics scripts plus 30%-sampled session recording, pulled fonts from two third-party CDNs, and
could fall back to public CORS proxies when importing a Google Doc — while `README.md` promised
"No External Services" and "No Data Collection." This fork closes that gap. It is a difference of
*preference*, not a claim that upstream is wrong to fund the project the way it chooses: analytics
on a free web app is an ordinary trade, and the fork simply takes the other side of it.

## What changed

**[CHANGES.md](CHANGES.md) is the authoritative log** — every change this fork makes to upstream,
dated and newest first, with a note on whether it is worth cherry-picking. It is not duplicated
here; this file explains *why*, that one records *what*.

Two entries stand on their own merits regardless of what you think of the privacy work: the blog
generator's CRLF fix, which silently degraded the SEO of 26 pages for any Windows contributor, and
the font self-hosting, which removes two CDN dependencies while keeping the OpenDyslexic
accessibility option working.

## The files

| File | What it is |
|---|---|
| [CHANGES.md](CHANGES.md) | The fork's changelog. Every change gets an entry here. |
| [2026-08-05-state-of-project.md](2026-08-05-state-of-project.md) | Dated assessment of the codebase **before** any change. Partly superseded — see its banner. |
| [work-scout.md](work-scout.md) · [work-scout.json](work-scout.json) | The candidate work ledger, with a PCFIRG header and a counter-argument per item. The JSON is the machine-readable copy. |
| [privacy-hardening.md](privacy-hardening.md) | The prompt that implemented W1–W4. Already executed; kept as a record and as a reusable recipe. |

The conventions these files follow — and the build gotchas that bite anyone editing this repo —
are written up for agents in [`AGENTS.md`](../../AGENTS.md) at the repo root.

## Reading these fairly

The assessment is candid about gaps between the upstream docs and the upstream code, because that
was its job — it was written to decide whether this codebase was safe to run locally, and it
concluded that it was. Two things it found are worth stating plainly here so they are not
mistaken for something worse:

- **No malicious code was found.** The dependency lockfile is fully registry-pinned with integrity
  hashes, there are no install hooks, no obfuscation, and the build scripts do nothing but local
  file I/O. The upstream project is safe to clone and build.
- **The tracking was ordinary product analytics**, self-hosted on the author's own infrastructure,
  not covert data collection. The criticism is that the README's privacy claims had not kept pace
  with it — a documentation gap, which is common and easily fixed.

Every factual claim in these notes cites a `file:line` that was actually read. Where something
could not be verified from source — the behavior of a remote script, or a deployed Cloudflare
Worker — the notes say so rather than guessing.

## For agents working in this repo

Start with [`AGENTS.md`](../../AGENTS.md) — it carries the conventions, the invariants, and the
build gotchas. Then read [work-scout.md](work-scout.md) for open work; W5 is the only item still
open. Do not re-run [privacy-hardening.md](privacy-hardening.md) — it is finished, and its line
numbers refer to files as they were before it ran. Treat the state doc as history, not as a
description of the current tree.

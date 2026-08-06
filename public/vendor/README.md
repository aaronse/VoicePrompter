# Vendored web fonts

Self-hosted so that no page in this fork makes a request to a third-party font CDN.
Every HTML entry point links `/vendor/fonts.css` instead of `fonts.googleapis.com`,
`fonts.gstatic.com` and `fonts.cdnfonts.com`.

Fetched 2026-08-06.

## google/dm-sans

Google Fonts DM Sans v17 — the same variable `woff2` files the `css2` API serves for
`family=DM+Sans:ital,opsz,wght@0,9..40,300..600;1,9..40,400`. All the weights the site
uses (300–600) come from one variable file per style/subset, so there are four:

| File | Source |
| --- | --- |
| `dm-sans-latin.woff2` | `https://fonts.gstatic.com/s/dmsans/v17/rP2Hp2ywxg089UriCZOIHQ.woff2` |
| `dm-sans-latin-ext.woff2` | `https://fonts.gstatic.com/s/dmsans/v17/rP2Hp2ywxg089UriCZ2IHSeH.woff2` |
| `dm-sans-italic-latin.woff2` | `https://fonts.gstatic.com/s/dmsans/v17/rP2Wp2ywxg089UriCZaSExdy3sGt9zz86GPwyKy58Q.woff2` |
| `dm-sans-italic-latin-ext.woff2` | `https://fonts.gstatic.com/s/dmsans/v17/rP2Wp2ywxg089UriCZaSExdy3sGt9zz86GPwyKK58VXh.woff2` |

Licensed under the SIL Open Font License 1.1 — see `google/dm-sans/OFL.txt`.

## opendyslexic

The four `OpenDyslexic` faces from `https://fonts.cdnfonts.com/s/19808/` (the family
`src/render.ts` selects for the accessibility font option). The `OpenDyslexic3`,
`OpenDyslexicAlta` and `OpenDyslexicMono` faces that CDN also serves are unused and were
not vendored.

Licensed under the SIL Open Font License 1.1 — see `opendyslexic/LICENSE.txt`.

## Refreshing

Re-download the URLs above with a desktop browser `User-Agent` (Google's `css2` endpoint
serves `ttf` instead of `woff2` to unrecognized agents), then bump the version note here.
If DM Sans moves past `v17`, re-fetch the `css2` stylesheet to get the new file URLs and
`unicode-range` values and update `fonts.css` to match.

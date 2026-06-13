# World Cup 2026 — Where & When to Watch

A single-file, zero-dependency dashboard that answers one question for a US viewer: **for every 2026 FIFA World Cup match, when does it kick off (Eastern), what channel is it on, and where can I stream it?**

All 104 matches, June 11 – July 19, 2026, hosted across the USA, Canada, and Mexico.

![Status](https://img.shields.io/badge/matches-104-1FB87A) ![Dependencies](https://img.shields.io/badge/dependencies-0-2DE89A) ![Build](https://img.shields.io/badge/build-none-F0B33A)

## Highlights

- **Every match, grouped by day** with sticky date headers; auto-scrolls to today on load.
- **English & Spanish TV** per match — FOX/FS1 and Telemundo/Universo chips on every row.
- **Streaming** stated once (it's the same for all games): FOX One (English), Peacock (Spanish), Fubo (all).
- **Search** across team, venue, and round.
- **Filters** — round, team, and FOX-vs-FS1 dropdowns.
- **Toggles** — Today, Upcoming, and Watch Party mode.
- **Host quick-picks** — one-tap 🇺🇸 USA / 🇲🇽 Mexico / 🇨🇦 Canada chips (tap again to clear).
- **Live status** — pulsing Live, Full Time (with score), and Upcoming states, computed from the clock.
- **Calendar export (`.ics`)** — download upcoming matches that match your current filters, in two modes.
- **Responsive** down to mobile, with a quiet dark aesthetic.

## Quick start

There is nothing to build or install. Either:

```bash
# Open directly
open index.html            # macOS

# …or serve the folder
python3 -m http.server
# then visit http://localhost:8000
```

The only external dependency is Google Fonts (loaded via `<link>`); everything else — markup, styles, data, and logic — is inline in `index.html`.

## Calendar export

Click **⤓ Add to calendar** to download a standard `.ics` file. The export always **excludes finished games** and **respects your active filters** — so filtering to USA and exporting yields only their remaining fixtures. Two modes, switched by the **Watch party** toggle:

- **Per-match** — one 2-hour event per game, with the venue as the location and both-language channels in the description.
- **Watch party** — one event per day spanning the first kickoff to the last match end, with every game that day listed in the description. Bundles a matchday into a single calendar block.

> If the in-frame download does nothing (sandboxed preview), open the page in its own browser tab.

## How it works

Everything lives in `index.html`: an inline `<style>` (dark theme via CSS custom properties), the markup, and an inline `<script>` that holds **both the data and the logic**.

- **`M`** — the match array, the single source of truth. Each entry is a compact object: `d` (date), `t` (display time, e.g. `"3:00 PM"` or `"FT"`), `sk` (intra-day sort key — the hour in 24h; `24` = post-midnight game listed under the prior day; fractional parts are tiebreakers, **not** minutes), `h`/`a` (teams, omitted for unresolved knockouts), `res` (score), `grp` (round), `v` (venue), `tv` (`"FOX"`/`"FS1"`), and `desc` (knockout matchup like `"1C vs 2F"`).
- **Derived at load** — `_st` (live/up/ft status) and `es` (Spanish channel: a hardcoded set of 12 group-stage games on Universo, the rest on Telemundo).
- **`state` → `passes(m)` → `render()`** — `state` holds all filters, `passes` decides visibility, `render` regroups visible matches by day and builds HTML via `card(m)`. Every control change triggers a full re-render.

### Time handling

All kickoff labels are **Eastern**. The clock math assumes **EDT (UTC−4)** because the tournament runs in summer; "today" is resolved in `America/New_York` so highlighting and auto-scroll are correct regardless of the viewer's timezone. Midnight (`12:00 AM ET`) games are shown under the prior matchday — matching FOX — but their real datetime rolls to the next calendar day in the `.ics` export.

## Data & provenance

Match data is **hardcoded**, not fetched — there's no stable, free per-match World Cup API with US broadcast assignments, and the schedule is fixed. The source-of-truth hierarchy:

- English channel (FOX vs FS1), kickoff times, and venues come from **FOX's published schedule** (the rights holder). Where sources disagree on English times/channels, FOX wins.
- The Spanish-language Telemundo/Universo split and the third-place match time come from a **secondary source** (FOX doesn't publish the Spanish split).

To update the schedule or scores, edit the objects in the `M` array. Adding a team requires a matching entry in `FLAGS` or it falls back to ⚽.

## Known limitations

- **EDT vs EST** — labels read "ET" and mean EDT; if literal EST is ever needed, shift the displayed labels and the `.ics` UTC math by one hour.
- **Spanish channel split** is from a secondary source — re-verify the 12-game Universo set if the network revises it.
- **Knockout matchups** show seeding (`1C`, `2F`, `3rd (…)`) until teams are confirmed; there's no logic yet to resolve them from group results.
- **No persistent storage** — `localStorage` is intentionally avoided (blocked in sandboxed previews); all state is in memory.

## Deploying to hatch.org

Live at **https://www.hatch.org/world-cup/**, served as a static asset from the
Hugo-based hatch.org blog (a separate repo, `hatch-org`, deployed via AWS Amplify).

This repo's `index.html` is the **source of truth**. Deployment is a copy step —
[`deploy.sh`](./deploy.sh) syncs it into the blog's static directory:

```bash
# edit index.html, commit here, then:
./deploy.sh            # copy into ../hatch-org/blog/static/world-cup/
./deploy.sh --commit   # copy AND commit it in the blog repo
# then push the blog repo → Amplify rebuilds the site
```

The blog's static dir defaults to `../hatch-org/blog/static/world-cup/`; override
with the `WORLD_CUP_DEST` env var if your layout differs. The copy at
`hatch-org/blog/static/world-cup/index.html` is **generated — never edit it
directly**; change it here and re-run `deploy.sh`.

## Possible next steps

- Resolve knockout bracket teams from entered group results (making `desc` dynamic).
- Optional live-score hydration from an API.
- A timezone selector (convert labels client-side; keep the `.ics` absolute).
- A second tier of "contender" quick-pick chips.

---

See [`HANDOFF.md`](./HANDOFF.md) for the full build rationale and [`CLAUDE.md`](./CLAUDE.md) for guidance when working in this repo with Claude Code.

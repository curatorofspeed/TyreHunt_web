# www.tyrehunt.app — Impeccable Audit & Polish Pass
**Scope:** car.html (new), lot.html, registry.html, spot.html · **Method:** bundled scan.sh + contrast.py on every text token against both surfaces (#0A0B0E page, #12141A panel), then live keyboard/DOM verification on a static server. Date 2026-09-04.

## Findings by severity

### 🔴 Critical
**No keyboard focus indication on lot.html and registry.html.** Zero `:focus-visible` rules; registry's search input also set `outline:0`, so a keyboard user tabbing the filter row or the search field saw nothing move. → **Fixed:** a shared floor block on all four pages — `:focus-visible{outline:2px solid var(--ember|--miami);outline-offset:3px}` plus a `@media (forced-colors: active)` variant switching to `CanvasText`; registry's search label gets `:focus-within` with the same ring (the input's own outline stays off so the ring wraps the whole field).

### 🟠 High
**Filter state lied to assistive tech (lot.html).** Six filter buttons toggled a visual `.on` class only; a screen reader heard six identical unpressed buttons. → **Fixed:** `type="button" aria-pressed` on each, kept in sync in the click handler; the row is a `role="group"` labelled "Filter the lot".
**Hero image alt lied (lot.html).** The hero swaps to the newest listing but kept a hard-coded "A Porsche 911 parked on the street". → **Fixed:** alt is set in the swap ("Latest on the lot: Jeep CJ-8 Scrambler").
**Search input 12px on phones (registry.html).** Below 16px iOS zooms the page on focus. → **Fixed:** `@media (max-width:760px){ .search input{font-size:16px} }`, placed after the `font:` shorthand so it wins the cascade (first attempt sat earlier in the sheet and lost — caught live at 375px, then re-verified at 16px).

### 🟡 Medium
**Silent status changes.** Loading → missing/empty swaps (car, spot, registry) and lot's "quiet card" rewriting itself to "The lot didn't load" were plain divs. → **Fixed:** `role="status" aria-live="polite"` on every element whose text changes after load.
**Motion ignored the OS.** registry's `.mcard` transition had no reduced-motion path. → **Fixed:** global `prefers-reduced-motion: reduce` kill-switch in the floor block (car/lot/spot have no motion; the rule is there so future motion inherits it).
**Unlabelled search field.** Placeholder-only input. → **Fixed:** `aria-label`.

### 🟢 Low
**No brand `::selection`.** → **Fixed:** miami on all four (#18B7DC / #04222A, 6.98:1).
**Most Wanted list unlabelled (car.html).** → **Fixed:** `aria-label="Most Wanted, ranked by sightings"` on the board container.

### Measured and left alone (correctly)
Every text token passes AA on both surfaces — ink 17.7/16.6, muted 7.46/6.98, miami 8.29/7.75, judge 7.97/7.45, gold 12.3/11.5, ok 9.86/9.22, chips everyday 6.36, classic 8.35, rare 6.88, podium bronze 6.81, CTA text on gradient 6.98–9.88, price pill text on --ok 8.30. No token changes were needed, so the visual identity is untouched. Decorative borders/dividers are WCAG-exempt and unchanged.

## Verified (live, static server, real key events)
- lot.html: mouse-click "All" then a **real Tab** → "Legendary" reports `:focus-visible` true, computed `outline: solid 2px rgb(24,183,220)`, offset 3px — screenshot shows the ring, no ring after the mouse click.
- lot.html: activating a filter flips `aria-pressed` across all six (All false, Legendary true) and `.on` stays in sync; `#filters` reads `group / Filter the lot`; `#quiet` reads `status / polite`; hero alt reads "Latest on the lot: Jeep CJ-8 Scrambler".
- car.html: click into page then real Tab → "Name your car" link `:focus-visible` true, same computed ring; `#loading` `status/polite`, `#bEmpty` `status`, board `aria-label` present.
- spot.html (?c=78, live data): `#loading` `status/polite`, `#missing` `polite`, photo `aria-label="Photo of Jeep CJ-8 Scrambler"`.
- registry.html at 375×812: search input computed `font-size: 16px`; focusing it gives `.search` `:focus-within` true with computed `outline: solid 2px rgb(24,183,220)`; `#loading` `status/polite`; input `aria-label` present.
- All four inline scripts parse after every edit (scan.sh).

Preview-pane notes (not product bugs): synthetic Enter/Space in the pane did not activate a native `<button>` (browsers do this natively; a dispatched click proved the handler), and real clicks time out while the pane is hidden — `:focus-within` was verified by programmatic focus, which is valid for that pseudo-class.

## Recommended (not done)
- **index.html, lobby.html, privacy/terms** were outside this pass; the same 6-line floor block applies verbatim and should be pasted into each.
- **Populated car page** (car.html?id=) was verified against the RPC's JSON shape only — no Star Car exists yet. Re-run the Tab pass over the sightings grid once one does.
- registry.html nav "current page" marking (`aria-current`) — the `.on` link markup didn't match the anchor, left for a manual look.

---

# Meet board and the operator's Meets tab — 2026-09-26
**Trigger:** Drew wants Cars & Coffee meets to play Tyre Hunt ("Meet Mode"; the app side is in the app repo's AUDIT.md).
- **`lobby.html` is the meet board.** States from `public_lobby` v2: upcoming (start date and time, how to play), live (countdown, standings by Show Score with ★ firsts, awards so far, the photo grid with FIRST HERE marks), closed (provisional podium and standings until an hour after the close, then final). The join band points at `hunt.tyrehunt.app/?lobby=CODE`. Storage paths are encoded segment by segment before going into `url()`. The quality floor is pasted in. There is no whole-board live region: a short "Leader: @x, N points" line is announced when the leader changes.
- **`?tv=1`** for a tablet or screen at the organizer's table. It shows standings (only the rows that fit), a SCAN TO PLAY QR plus the code and address, a countdown, the photo strip, a Screen Wake Lock and a fullscreen button where supported. A rare or legendary first takes the screen for 10 s: at most one per 3 minutes, only finds under 5 minutes old, never on the first load or after the close. Portrait layout for a phone held upright. Bad codes and load failures show on the screen. The QR is drawn only for a loaded meet.
- **`admin.html` Meets tab.** List (state, checked, pin, hunters, counted shots, RSVPs, BOARD and TV links) and a form: name, place, start/end in local time, pin from "lat, lng" or a Google Maps link (a dropped pin preferred, with a warning when a link only carries the map view centre), fence radius 50–2000 m, organizer Instagram (normalised), Checked meet. `admin_hunts` / `admin_hunt_save` refuse non-operators; new meets get a code free in both hunts and old events.
**Verified:** board harness with stubbed data in live, closed, provisional, final and upcoming states; TV portrait fits with no overflow after fonts load; a new legendary first takes the screen for 10 s and clears; a missing code shows on the TV with no QR. Pin parser and Instagram normaliser unit-tested; admin RPCs tested in rolled-back SQL. Two adversarial reviews (see the app repo's AUDIT.md).

---

# The judge moves to Claude Sonnet 5.5 — 2026-09-28
**Trigger:** Drew: "sonnet 5.5 is out… should we upgrade it?" then "move forward with your plan" (side-by-side test, switch with an instant rollback, watch a week). The judge is the `verify` edge function; this repo only gains the console's watch panel.

## The side-by-side (39 stored capture photos, temporary function `judge-eval`, now deleted; results kept in `judge_eval_runs`)
- **Arms:** Sonnet 5 as live (thinking disabled); Sonnet 5.5 with thinking `between_tools` (its only "off"; 5.5 rejects "disabled", Sonnet 5 rejects "between_tools"); Sonnet 5.5 with structured outputs on the capture verdict. Every capture photo twice per arm, plus framing, the one for-sale sign and four hunt plans. 327 calls, $3.18.
- **Format:** Sonnet 5.5 without a schema broke the JSON at the end of the blurb in 16 of 78 answers (once it wrote a draft and a "correction"). With structured outputs: 0 of 78. Sonnet 5: 0 of 78.
- **Speed:** capture p50 3.5 s (Sonnet 5) → 2.9 s (Sonnet 5.5 + schema); framing, sign and plans also faster.
- **Blind review** (3 reviewers, answers labelled X/Y, sides alternated): disputed identifications 12–1 for Sonnet 5.5 (Studebaker Lark vs "Mercedes W123", Harley Softail vs "Vespa", BMW M4 GT4 vs "Toyota GR86 Cup", Jeep CJ-8 vs "CJ-7", 1932 Fords vs "Model A"; Sonnet 5 won one: "911" over an over-specific "911 SC"); rarity 4–1; car boxes 14–1; plate boxes 7–0 (one small plate missed by both).
- **Found and fixed before the switch:** with structured outputs, Sonnet 5.5 misreports `image_w/h` (20 of 78) while its coordinates stay in real pixels, which would have misplaced plate and car boxes — boxes are now scaled by the photo's measured size. `verify_receipts.model` is the *car* model; the AI model goes in the new `judge_model` column (a test caught the collision).
- **Watch items:** Sonnet 5.5 flipped rarity between two runs of the same photo 3 times in 39 (Sonnet 5: 0) and varies trim wording more (8 vs 4). Cost per capture $0.0116 → $0.0149 (the schema's format instructions add ~1,550 input tokens), about $0.20 more a month at today's volume, about $4 more for a 1,250-photo meet.

## What changed (verify v42, deployed; v41 kept)
One `askClaude` helper for all four calls: the model and its matching thinking setting from `JUDGE_MODELS`, structured outputs on the Sonnet 5.5 capture verdict, `stop_reason` checked (a refusal or cut-off is a 502 the app retries), one `judge_calls` row per call. A reply that can't be parsed as its mode's JSON fails rather than salvaging an inner object. Enum values compared lowercase. Receipts (capture and sign) record `judge_model`.
**Rollback:** set the function secret `JUDGE_MODEL=claude-sonnet-5` (an unrecognised value also falls back to Sonnet 5, with a log line); full code rollback is redeploying v41 (`~/Claude/TyreHunt-judge-backups/verify.v41.ts`).
**Console:** the Judge tab shows a Model watch: per model, captures judged, corrected, rare share and confidence; per day, calls, failures and why, p50/p95 latency and tokens (`admin_judge_watch`).
**Verified:** live API accepted both models' request shapes in the side-by-side; local harness drove all four modes through the compiled v42 with stubs (23 checks × default, rollback, mistyped and junk settings); an adversarial review of the diff found 5 issues, all fixed and covered by tests; deployed source matches, bad tokens still 401, no boot errors.
**Not done:** no real hunter photo has gone through v42 yet; the first one will show in the watch panel. The race-roster reader stays on Sonnet 5 for now; the Car of the Day writer stays on Opus 5; FishDex is separate.

---

# Cars & Coffee organizer page (meets.html) — 2026-09-29/30
**Trigger:** Drew: "make the website highlight how Tyrehunt can gamify their cars and coffee event". Built Sep 29, lost unpushed when the session scratchpad was wiped, rebuilt byte-identical from the session transcript Sep 30, then reviewed.
- **meets.html** (homepage stylesheet inlined + page CSS): hero with an EXAMPLE rare-find takeover; how it plays; the scoring ladder (1/3/5/10, FIRST HERE +2, never "points"); what you get (poster, TV board, live board, results); an EXAMPLE TV board (DURING / AFTER THE CLOSE, takeover preview) that shows only what the real TV draws; fair play; two setup paths (in the app: MEETS → HOST A MEET → PIN IT AT THE LOT → PRINT POSTER → TV BOARD; or the request form); FAQ + FAQPage JSON-LD. Every claim checked against the app, lobby.html and the database; copy source and "never claim" list are in the working notes.
- **Request form** → Supabase `meet_requests` (anon column INSERT; RLS; trigger `meet_requests_brake`: 3 per contact per day, 120 per hour overall, each with its own hint so the page says which; the busy case points to hello@curatorsofspeed.com). Honeypot, local-date minimum, end-after-start, busy guard, aria-live status.
- **index.html:** "Cars & Coffee" nav link; nav kept to one line at every width (links drop by priority p5→p1: Star Cars, How it works, The Lot, Car of the Day, Hunter's Edition; the nav row widens to 1320px); a "Run a Cars & Coffee? Make it a game." band; footer link; the phone mock's tab bar now matches the app (MEETS).
- **privacy.html:** meet boards are public (photos on the lot during the meet, car, rarity, handle; never location or email); what the request form collects and how long it's kept. **admin.html:** Meets tab lists organizer requests (escaped), MARK HANDLED / REOPEN. **lobby.html:** "pts"/"PTS"/"points" removed (the site never says points). **sitemap.xml:** meets.html.
**Review (4 reviewers + 4 skeptics):** fixed: takeovers are rationed on the real TV (copy now says "can", one every few minutes); the example board drew award tiles and a podium the TV doesn't have (now stats, standings kept after the close); Path A had no way to the TV board (app gained TV BOARD); offline shots only count for hunters already on the board; codes are 4 characters, not letters; checked-meet copy now says a re-upload of the same picture still counts as a copy; privacy wording; takeover no longer steals focus, covers its board with `inert`; anchors clear the sticky nav; the nav CTA shortens under 360px; lot field no longer autofills a home address; validation names the actual problem; the global brake can't be cheaply filled.
**Verified:** preview: board states, takeover open/Esc/timer without focus theft, form messages (stubbed fetch incl. both brake hints), no horizontal scroll 320–1440 px on meets/index/privacy, nav one line at every width; brake tested in a rolled-back SQL transaction.
**Not done:** no real request has been submitted from the live page; no meet has run.

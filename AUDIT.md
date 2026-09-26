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

# PROGRESS — FPV Visual Wiring Schematic Builder

Single source of truth for what is **done**, what is **being verified**, and what **must still be built**
(design spec: [`DESIGN_RULES.md`](DESIGN_RULES.md)).

Legend: `[x]` done · `[~]` partial / needs verification · `[ ]` not started

---

## 1. Done

### Core app
- [x] Single self-contained `index.html` (vanilla JS/CSS, no build step), GitHub Pages ready
- [x] Canceled pointer gestures discard in-progress wires and restore transient pan/drag changes; `.fpv` imports reject malformed component/wire entries before hydration
- [x] Pure SVG canvas: pan (middle-click / spacebar / wheel), zoom 10–500%, reset
- [x] Edit / Preview modes
- [x] Backgrounds: Dark Grid (default), Light Grid, Blueprint, Clean White, Transparent
- [x] Header tagline tracks current scope (`Parametric parts · routed & styled wires ·
      .fpv / SVG / PNG export`) — stale "Phase 3 — wiring engine…" text removed
- [x] `localStorage` persistence (`fpv_builder_phase1_v1`) with save migration/backfill
- [x] Browser verification on GitHub Pages (2026-10-10): malformed component/wire imports are rejected without changing the board; `.fpv` export → import restores 10 parts and 4 wires; SVG and PNG downloads complete; no browser console errors. Pointer-cancel and new-wire drag gestures were not reproduced by browser automation and remain unverified.

### Snapping
- [x] Component body grid snap: 1 / 2 / 5 mm / Off
- [x] Magnetic pad snap (15 px) with pulsing snap halo
- [x] Stacked-pad dodge so adjacent-row wires stay separate (4.5 px lead-out)

### Parametric components
- [x] Board W/H, per-corner styles (square / round / chamfer, per-corner size)
- [x] Mounting-hole presets (12x12…30.5x30.5, custom) + diameter
- [x] Rotation ±90° buttons + free-angle input; Flip H / Flip V; upright name labels
- [x] Pad visibility toggles (sparse FC silkscreen), pad editing (add/delete/rename/type)
- [x] Connectors: JST-SH, GH1.25, XT60, XT30, USB-C with pin snap targets
- [x] Per-edge ordered pad/connector groups + drag & drop reordering
- [x] Pad label side / rotation, component name placement (top/bottom/left/right/center)
- [x] Per-group **Fit** (auto pad pitch when board size changes); default pad gap **3 mm**
- [x] Built-in library: Flight Controller, 4-in-1 ESC, Analog VTX, ELRS RX, Analog Camera
- [x] Blank **Custom Part** (30×30, no pads/holes) offered in the Library add-part menu only —
      never seeded into the default test layout; shape it from the properties panel
- [x] **Motor** part: Ø30 round body (corner radius = size/2), 3 phase pads (1/2/3) on the
      bottom edge; **`MTR`** signal type also applied to the ESC `Motor 1..4` phase groups
- [x] Default test layout: 4 Motors flank the ESC (2 left / 2 right, ±90° so phase pads
      face it) — `Motor 4/3` left = ESC `Motor 4/3` groups, `Motor 2/1` right likewise
- [x] Pad edge inset made uniform: ESC `Motor 1..4` + Motor `Phases` groups dropped their
      stray `cross:4` lane (5.5 mm) → standard **1.5 mm** from the edge, labels outside,
      like every other part (FC's stacked Row 2 keeps `cross:4` on purpose)
- [x] Templates verified against hardware spec (FC pad order, `fc_check.py`)
- [x] **FC back side (spec §3.2 dual-side)**: linked companion part `fc_back` auto-created
      with every FC (seed, +Add, old-save migration) — independent placement, paired
      delete, panel "Paired" indicator; pads 4-3-2-1/T/C/G/V left-aligned on top edge

### Wire routing & styling
- [x] Pad-to-pad wire drawing with snap termination; unlimited wires per pad
- [x] Auto wire color from `signal_type` (full color-pipeline table) + manual color picker
- [x] Gauge select (12–30 AWG, manual) — **default 30 AWG**, **20 AWG for `MTR` pads**
- [x] Line patterns: solid / dashed / striped
- [x] Bézier route + **orthogonal (right-angle)** route (default)
- [x] Obstacle avoidance: A* on Hanan grid routes wires **around** parts (`ROUTE_CLEAR` 6 px)
- [x] Fallback: unroutable wires drawn **beneath** part bodies (`#layerWiresUnder` layer)
- [x] Corridor resolution + lane stacking for parallel wires; lane gap **6 px**
- [x] Wire-vs-wire separation on A* routes (already-routed wires as soft obstacles)
- [x] "Appendix" spike fix: stubs clamped to corridor; `extendOut` jumps exactly to boundary
- [x] Configurable **Lead-out** setting (toolbar, mm, default **15 mm**, persisted)
- [x] **180°-rotation "bump" fix**: `sin(π)` float residue (~1.2e-16) in rotated pad
      directions passed `dir.x !== 0`, so vertical wire pairs took the mixed-corner
      branch and the lane offset became a ~100 px diagonal — now classified by
      dominant axis + directions quantized in `dirToWorld()` (verified on user's
      `.fpv` export: all 19 wires orthogonal, loom lanes stacked at 6 px)
- [x] Layer split: wires-under / boards / wires / pads / overlay

### Files & exports
- [x] **Project file `.fpv` export** — downloads full project JSON (state + embedded templates)
- [x] **Project file import** — file picker, validation, full migration via shared `hydrateState()`
- [x] **SVG export** — standalone drawing (content bbox, page CSS embedded, no grid/UI chrome)
- [x] **PNG export** — same view rasterized at 2x via canvas

### Verification tooling
- [x] `jscheck.py` — JS bracket/token balance (passes; last run 2757 lines)
- [x] `fc_check.py` — FC pad order vs spec
- [x] `lane_check.py` — lane assignment
- [x] `route_check2.py` — base-route lanes, appendix regression, A* soft separation

### Docs
- [x] `DESIGN_RULES.md` — repo context (Part A) + design spec (Part B) + backlog note
- [x] Documentation sync — `.fpv` example matches the version 1.4 export/import shape; export capabilities, library status, CSS dependency guidance, and file-size note match the current app/repo constraints
- [x] `PROGRESS.md` — this file

---

## 2. Needs verification (in browser)

- [ ] Latest batch works after reload: wires render, **15 mm lead-out**, no appendix spikes,
      **6 px** lane spacing looks right, **30 AWG** default visible in wire panel
- [ ] Existing `localStorage` may still hold old defaults (lead-out `10`, gauges `18/26`,
      `route:'bezier'`) — stored values are never overwritten by new defaults
- [ ] Mixed-corner branch (`horizA ⊕ horizB`, `wirePathOrtho`) has **no** overshoot clamp —
      appendix could still appear there
- [ ] New File toolbar: test `.fpv` export → import round-trip, `.svg` opens standalone,
      `.png` rasterizes at 2x with correct background

---

## 3. Must be done (from design spec)

- [ ] **Validation engine** (spec §5.2): UART swap check (TX→TX / RX→RX), short-circuit alert
      (power→ground), missing-ground highlight for Camera/VTX/RX
- [ ] **Junction nodes / Y-splits** (spec §4.2): double-click a wire to spawn a branch node;
      must also unlock **junction color inheritance** (§5.1)
- [ ] **Manual segment waypoints** (spec §4.1): click canvas to place intermediate anchors
- [ ] **Bézier interactive control handles** (spec §4.1)
- [ ] **Print / PDF workbench view** (spec §6) — SVG/PNG export done, print view not
- [ ] **Save as Template** (spec §6): store custom boards in browser storage
- [ ] **Library expansion**: GPS, BEC, Buzzer templates (spec lists 8; we ship 6 + blank Custom Part)
- [ ] **Grid snap for wire anchor points** (spec §2.1) — currently only component bodies snap

---

## 4. Explicitly deferred to later stages

- [ ] **Wire gauge auto-select** from signal type / current draw — `defaultGauge()` returns a
      fixed 30 AWG (TODO marker in code + `DESIGN_RULES.md` §Later stages backlog)

---

## 5. Nice-to-have / polish

- [ ] Re-tune lane gap (6 px) and `ROUTE_CLEAR` (6 px) after visual review (±2 px)
- [ ] A* output for wires that already have lane offsets (soft obstacles approximate this)
- [ ] Validation for connector pins vs signal types

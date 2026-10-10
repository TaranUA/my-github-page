# AGENTS.md

## What this repo is

- Single-file static app: everything (HTML/CSS/vanilla JS/SVG) lives in root `index.html` (about 3.2k lines; count changes as features are added). No `package.json`, no build, no Node, no CI, no tests, no lint. Do not add a toolchain or CDN deps unless explicitly asked.
- Deploys via GitHub Pages on push to `main`; root `index.html` must stay the entry point.
- Spec: `DESIGN_RULES.md` (Part A = repo rules, Part B = FPV wiring-builder feature spec + `.fpv` JSON schema). Status/backlog: `PROGRESS.md` — it is the source of truth for done/verified/remaining; update it when you complete or verify work.
- Keep documentation in UTF-8. `DESIGN_RULES.md` uses the current `.fpv` export shape; update its schema example when the export/import format changes.

## Verification (there are no tests)

- Only real check: open `index.html` in a browser and exercise the changed feature (Edit/Preview modes, wire drawing, export/import).
- Python is available for ad-hoc scripts. `jscheck.py` / `fc_check.py` / `lane_check.py` / `route_check2.py` are mentioned in `PROGRESS.md` but are **not in the repo** — don't look for them; recreate only if needed.
- Node is not installed — `node --check` etc. will fail.

## `index.html` map

The `<script>` is divided into 12 numbered `====` sections (line numbers shift; navigate by name):
1 Template library · 2 Application state · 3 Geometry helpers · 4 Colour pipeline · 5 Viewport · 6 Render pipeline · 7 Snapping · 8 Input handling · 9 Toolbar wiring · 10 Test fixture + persistence · 11 Component properties panel · 12 Boot. CSS has its own 5-section comment header.

- Coordinates: boards authored in mm, scaled by `SCALE = 3` px/mm; screen = `pan + world * zoom`. `SNAP_RADIUS_PX = 15` is in **screen** px.
- SVG layer order (z): `layerWiresUnder` → `layerBoards` → `layerWires` → `layerPads` → `layerOverlay`. Unroutable wires intentionally render *under* boards — keep that behavior.

## Persistence & migration (easy to break)

- localStorage key `fpv_builder_phase1_v1`, debounced 300 ms save.
- `hydrateState()` is the **single migration path** for both localStorage loads and `.fpv` file imports. Any new persisted field must get a backfill there (see how `gridSnap`, `wireLeadoutMm`, wire `gauge/pattern/route` are handled) and be added to `serializableState()`.
- Defaults are backfilled on read only — old stored values are never overwritten. To test fresh defaults, clear the storage key first (browser devtools), otherwise old values (lead-out 10, gauges 18/26, `route:'bezier'`) mask them.
- `state._targets` is derived during render — never serialize it.
- Each placed component embeds its own cloned `template`; old template formats are upgraded by `migrateTemplate()` (legacy `pads`/`connectors` → `edges` group model). New template fields need a backfill there too.
- Every FC auto-creates a paired `fc_back` companion (`pair_id` on both, see `linkedPair()`/`addPairedBack()`); `hydrateState()` spawns it for old saves. Preserve pairing on delete/migrate.

## Conventions that differ from defaults

- Panel/toolbar UI is built as template-string HTML with delegated events: `data-action` attributes routed through `handlePanel()` / `onPanel*`. Add new controls via that pattern, not per-button listeners. Escape user text with the existing `esc()` helper.
- Key constants: `ROUTE_CLEAR = 6`, `WIRE_LANE_GAP = 6`, lead-out default 15 mm, wire gauge default **30 AWG for all new wires** — the fixed default is a deliberate backlog item (`defaultGauge()` TODO), not a bug to "fix".
- Pad/connector layout uses an ordered per-edge group model (`padsGroup`/`connGroup`); FC boards use sparse pad visibility by default.
- `.fpv` files are plain JSON in the current shape documented in `DESIGN_RULES.md` §7 (export version `1.4`); import goes through `hydrateState()`, so round-trip `export → import` is the regression check for state changes.

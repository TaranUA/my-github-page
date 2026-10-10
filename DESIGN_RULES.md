# AI Agent Rules: Repository Context & FPV Wiring Schematic Builder

## Part A - Repository Context & AI Agent Instructions

### Project Overview
- **Repository Name:** `my-github-page`
- **Owner / Account:** `TaranUA`
- **Deployment Platform:** GitHub Pages (Static Hosting)
- **Live URL:** `https://TaranUA.github.io/my-github-page/`
- **Primary Use Case:** Dynamic testing sandbox for LLM/AI code generation, UI experiments, and frontend feature deployments.

---

### Technical Constraints & Environment

1. **Client-Side Only (Static Environment):**
   - No backend runtime (Node.js, Express, Python, PHP, etc.).
   - No native database server (PostgreSQL, MySQL, MongoDB).
   - Any API interactions must be client-side REST/GraphQL calls (`fetch()` / `axios`).
   - Browser `localStorage` / `sessionStorage` or external APIs are used for data persistence.

2. **File Structure & Routing:**
   - Entry point must remain **`index.html`** in the repository root directory (`/`).
   - Single-file SPA structures are preferred for quick testing, but relative linking to secondary static files (`/assets/`, `/css/`, `/js/`) is permitted.
   - SPA routing relies on hash routing (`/#/route`) or client-side history API.

3. **Deployment Trigger:**
   - Deploys automatically via GitHub Pages on every push to the **`main`** branch.
   - Build delay is typically 1–2 minutes.

---

### Instructions for AI Agents Editing This Repository

When generating, modifying, or replacing code in this repository:

1. **Maintain Standalone Integrity:**
   - Keep `index.html` self-contained unless external modular files are explicitly requested.
   - Use clean inline CSS. Do not add a CDN dependency or CSS framework unless explicitly requested.
2. **Preserve Deployment Readiness:**
   - Do not break the root `index.html` entry point.
   - Avoid requiring build steps (like `npm run build`) unless a GitHub Actions workflow is explicitly configured for it.
3. **Interactive Features Guidelines:**
   - Use `localStorage` for state preservation across page reloads.
   - Keep UI responsive and mobile-friendly.

---

## Part B - FPV Visual Wiring Schematic Builder Design Rules

This section describes the target feature set. The implementation status is tracked in [`PROGRESS.md`](PROGRESS.md); items listed here are not necessarily implemented yet.

### 1. Core Principles & Application Modes

- **Physical Realism First:** Components visually resemble real FPV hardware (exact board shapes, corners, holes, active pads, connectors) to guide physical workbench soldering.
- **Pure SVG Engine:** Render all components, pads, connectors, wires, and junctions on an interactive vector canvas (optimized for 10–15 components max).
- **Application Modes:**
  - **Edit Mode:** Active grid overlays, component dragging, parametric dimension editing, pad/connector management, and interactive wire routing.
  - **Preview Mode:** Clean presentation layout hiding all UI guides, grids, and handles. Optimized for workbench viewing, printing, and image export.
- **Background Modes:** Grid Dark Mode (default), Grid Light Mode, Blueprint Mode, Clean White, and Transparent (PNG/SVG export).

---

### 2. Canvas, Viewport & High-Priority Snapping

#### 2.1 Viewport Mechanics
- **Pan & Zoom:** Infinite canvas navigation via middle-click drag, spacebar drag, or mouse wheel zoom ($10\% \to 500\%$).
- **Grid Snapping System:** Configurable snapping ($1\text{mm}$, $2\text{mm}$, or $5\text{mm}$) for positioning component bodies and intermediate wire anchor points.

#### 2.2 Magnetic Pad Snapping (High Priority)
When drawing or dragging a wire segment:
- **Pad Magnetic Target Radius ($R = 15\text{px}$):** Replaces background grid snapping whenever the cursor approaches an active solder pad, connector pin, or junction node.
- **Visual Feedback:** Target pads highlight with a pulsing glow ring when within magnetic snap distance.

---

### 3. Parametric Component & Connector Engine

#### 3.1 Board Geometry & Per-Corner Configuration
- **Outer Profile:** Width ($W$), Height ($H$), and individual corner styling.
- **Flexible Per-Corner Styles:**
  - Board corners (`top_left`, `top_right`, `bottom_right`, `bottom_left`) can be configured **individually** or set via a global "all-corners" preset.
  - Each corner independently supports:
    - `type`: `square` | `round` | `chamfer`
    - `size`: Radius $r$ or chamfer offset $d$ in millimeters.
  - *Use Case:* Supports asymmetrical boards (e.g., clearance chamfers on the front edge and rounded corners on the rear edge).
- **Mounting Hole Patterns:** Presets (`20x20`, `30.5x30.5`, `25.5x25.5`, `16x16`, `12x12`, custom) with configurable hole diameter ($M2 / M3$).

#### 3.2 Transformation & Dual-Sided Views
- **Rotation:** $90^\circ$ incremental rotation or free angle adjustment.
- **Flip / Dual-Sided Views:** Flip component horizontally/vertically. Secondary board face pads can be rendered as linked component views or toggled via pad visibility.

#### 3.3 Pad Efficiency & Physical Connectors
- **Sparse Pad Selection (For Complex FC Boards):**
  - Users only toggle visibility for pads actively used in the build (plus spares) to keep the FC silkscreen uncluttered.
  - Small boards (ELRS RX, GPS, VTX) display all pads by default.
- **Visual Connectors & Sockets:**
  - Distinct physical silhouettes for connectors: `jst_sh_socket`, `gh1_25_socket`, `xt60_male`/`xt30_male`, `usb_c_port`.
  - Pins inside socket headers serve as magnetic snap targets for wire harnesses.

---

### 4. Smart Wire & Splice Routing Engine

#### 4.1 Interactive Routing Paths
- **Bézier Curves:** Smooth, organic curves with interactive control handles.
- **Orthogonal 90° Paths:** Automated right-angle routing around board geometry.
- **Manual Segment Waypoints:** Click anywhere on the canvas to place intermediate anchor points before terminating at a target pad.

#### 4.2 Multi-Wire Pads & In-Line Splices ($Y$-Harnesses)
- **Multi-Wire Solder Joints:** Attach $N$ wires directly to a single pad or connector pin without artificial limits.
- **Junction Nodes ($Y$-Splits):** Double-click any wire path to spawn a junction node ($X, Y$ canvas anchor) to split main power, ground, or signal lines into $N$ branch wires.
- **Auto-Routing Offset / Channel Bundling:** Automatic parallel offset gaps so overlapping parallel traces remain distinct and legible.

#### 4.3 Visual Wire Attributes
- **Gauge Styling:** Visual line thickness matching real wire gauges ($12\text{AWG}$ thick battery leads $\to$ $20\text{AWG}$ power $\to$ $28\text{AWG}$ fine signal lines).
- **Line Patterns:** `solid`, `dashed`, or `striped` (two-tone representation).

---

### 5. Color Pipeline & Intelligent Assist

#### 5.1 Smart Auto-Color Rules
When a wire is connected between pads, the app automatically assigns default wire colors based on pad `label` or `signal_type`:

| Signal Type / Label | Default Color | Description |
|---|---|---|
| `GND`, `ground` | `#1E1E1E` (Dark Black/Gray) | Ground return path |
| `5V`, `9V`, `BAT`, `VCC`, `power_pos` | `#E53935` (Red) | Power supply line |
| `CAM`, `VTX`, `VO`, `VI`, `video` | `#FDD835` (Yellow) | Video signal line |
| `TX`, `T1`, `T2`..., `uart_tx` | `#FFFFFF` (White) | Serial Transmit |
| `RX`, `R1`, `R2`..., `uart_rx` | `#4CAF50` (Green) | Serial Receive |
| `CRSF`, `SA`, `TLM`, `telemetry` | `#1E88E5` (Blue) | Telemetry / SmartAudio |
| `MTR`, `motor` | `#000000` (Black) | 3-phase motor wire (Motor + ESC motor pads) |
| Unmatched / General | `#FB8C00` (Orange) | Auxiliary / Buzzer / LED |

- **Junction Inheritance:** Branch wires splitting off a junction node automatically inherit the parent wire's color.
- **Manual Overrides:** Full color picker override per wire segment.

#### 5.2 Non-Blocking Validation Engine
1. **UART Swap Check:** Visual alert when `TX` connects to `TX` or `RX` to `RX` (recommending `TX` $\to$ `RX`).
2. **Short-Circuit Alert:** Red warning glow when a positive power line connects directly to a ground line or pad.
3. **Missing Ground Check:** Highlights active payloads (Camera, VTX, RX) that lack a completed ground path to the main power source.

---

### 6. Project & Template Management

- **Current built-in library:** Flight Controller, 4-in-1 ESC, Analog VTX, ELRS RX, Analog Camera, Motor, and a blank Custom Part. Adding an FC also creates its paired FC Back Side companion; that companion is not a separate library option.
- **Planned library additions:** GPS, BEC, buzzer, and additional VTX templates.
- **Planned custom template storage:** "Save as Template" to store custom board profiles in browser storage.
- **Current project format:** `.fpv` is a self-contained JSON project that stores placed components with their embedded editable templates, plus wire and view state. Its current exported shape is shown in §7.
- **Current drawing exports:** Standalone SVG and 2× PNG. Print/PDF workbench output is planned, not implemented.

---

### 7. Current `.fpv` Export Shape (version 1.4)

The app exports this top-level shape and imports files containing `components` and `wires` arrays. It does not currently use the older `app_state`, `library_templates`, `placed_components`, or `junctions` fields. Import runs through `hydrateState()`, which backfills defaults and migrates older component templates. `version` is written on export but is not currently checked by the importer.

Component `x`/`y` coordinates are world-space pixels; template dimensions and pad layout values are millimetres. Each placed component embeds its own template copy, whose pads are represented in ordered per-edge groups.

```json
{
  "app": "fpv-wiring-builder",
  "version": "1.4",
  "exported_at": "2026-10-10T12:00:00.000Z",
  "mode": "edit",
  "background": "dark_grid",
  "gridSnap": 5,
  "zoom": 1.0,
  "pan": { "x": 20, "y": 20 },
  "wireLeadoutMm": 15,
  "components": [
    {
      "instance_id": "vtx_1",
      "template_id": "custom_vtx_plate",
      "label": "Custom VTX Unit",
      "labelPos": "center",
      "x": 150,
      "y": 120,
      "rotation": 0,
      "flipX": false,
      "flipY": false,
      "template": {
        "template_id": "custom_vtx_plate",
        "name": "Custom VTX Unit",
        "width": 32,
        "height": 24,
        "corners": {
          "top_left": { "type": "chamfer", "size": 3 },
          "top_right": { "type": "chamfer", "size": 3 },
          "bottom_left": { "type": "round", "size": 2 },
          "bottom_right": { "type": "round", "size": 2 }
        },
        "mounting_holes": { "pattern": "20x20", "diameter": 2.1 },
        "edges": {
          "top": [
            {
              "id": "g_vtx_top",
              "kind": "pads",
              "align": "center",
              "spacing": 3,
              "cross": 0,
              "name": "",
              "labelSide": "",
              "labelRot": 0,
              "fit": false,
              "pads": [
                { "id": "pad_gnd", "label": "GND", "signal_type": "ground", "visible": true },
                { "id": "pad_vcc", "label": "BAT", "signal_type": "power_pos", "visible": true }
              ]
            }
          ],
          "right": [],
          "bottom": [],
          "left": []
        }
      }
    }
  ],
  "wires": []
}
```

Each wire entry stores an `id`, `from` and `to` endpoints (`comp` instance ID plus `target` pad/pin ID), `color`, `gauge`, `pattern`, and `route`.

### Later stages (backlog)

- **Wire gauge auto-select**: `defaultGauge()` returns 20 AWG for motor phase wires (`signal_type: 'motor'`) and a fixed 30 AWG for every other new wire (power, ground and signals alike). Selecting gauge automatically from signal type / expected current (thicker wire for power + ground, thinner for UART / control / video signals) must be implemented in a later stage.


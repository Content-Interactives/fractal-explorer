# Fractal Explorer

A **Create React App** (CRA) single-page app that renders **three classic fractals** on a **fixed-size HTML canvas** using recursive / iterative geometry: **Sierpinski triangle**, **Koch snowflake**, and **Heighway dragon**. Users pick a fractal tab and drive **iteration depth** with a range input.

**Live build (Content Interactives):** [https://content-interactives.github.io/fractal-explorer/](https://content-interactives.github.io/fractal-explorer/)

Curriculum, CK-12 links, and standards are summarized in [Standards.md](Standards.md).

---

## Stack

| Layer | Technology |
|--------|------------|
| Runtime | React 19, `react-dom` |
| Tooling | `react-scripts` 5 (Webpack, Babel, Jest entry via CRA) |
| Styling | Tailwind CSS 3 (`tailwind.config.js`, `postcss.config.js`, `index.css`) + utility classes in `FractalExplorer.jsx` |
| Graphics | **Canvas 2D** (`getElementById('fractalCanvas')` + `getContext('2d')`) |
| Deploy | `gh-pages -d build` (`predeploy` runs `npm run build`) |

---

## Repository layout

```
package.json              # homepage field controls CRA asset base path for production
public/index.html         # CRA shell
src/
  index.js                # ReactDOM.createRoot → <App />
  App.js                  # Renders <FractalExplorer />
  FractalExplorer.jsx     # All fractal logic, canvas drawing, UI (~475 lines)
  index.css, App.css
tailwind.config.js
postcss.config.js
```

There is **no** separate `js/` or `css/` tree at repo root; logic lives under `src/`.

---

## Application architecture (`FractalExplorer.jsx`)

### State

- **`activeFractal`:** `'sierpinski' | 'snowflake' | 'dragon'`.
- **`iterations`:** non-negative integer; slider **max 7** for Sierpinski and Koch, **max 12** for dragon.
- **`canvasSize`:** `{ width: 400, height: 400 }` — initialized once; **no resize handler** in current code.
- **`info`:** `{ elements, pattern, complexity, dimension }` updated inside each draw routine — **not bound to the UI** (dead state for display unless you add a panel).

### Render loop

`useEffect` depends on `[iterations, activeFractal, canvasSize]`. It clears the canvas, fills `#f8f9fa`, and `switch`es to:

1. **`drawSierpinskiFractal`** — Equilateral triangle in a margin box; **`drawSierpinskiTriangle`** subdivides via midpoints until `depth === 0`, then **`drawTriangle`** fills with palette color by depth. Reported triangle count: **`3^n - 1`** (with `n = iterations` in the `setInfo` copy).
2. **`drawSnowflakeFractal`** — Equilateral triangle on a circle; each edge is **`drawKochCurve`** (recursive thirding + equilateral bump). Segment count info: **`4^n × 3`**. Stroke width scales down for high `iterations`.
3. **`drawDragonFractal`** — **Heighway dragon:** builds a **turn sequence** (right/left) by the standard recursive-doubling rule, traces polyline in a local coordinate frame, **bounding-box centers** the path, applies **hand-tuned `zoomFactors`** per iteration so the curve stays on-canvas.

### Color

- **Sierpinski:** `colorPalettes.sierpinski` cycle by `depth % palette.length`.
- **Snowflake:** light fill + blue stroke `#0056b3`.
- **Dragon:** red stroke `#FF0000`; line width heuristics for visibility.

### Complexity strings

`getSierpinskiComplexity`, `getSnowflakeComplexity`, `getDragonComplexity` return canned prose by iteration index (capped by array length). Same **unused-for-render** situation as `info` unless wired to JSX.

---

## Scripts

| Command | Purpose |
|---------|---------|
| `npm start` | CRA dev server |
| `npm run build` | Optimized bundle → `build/` |
| `npm test` | Jest / RTL (default CRA) |
| `npm run deploy` | Build, then publish `build/` to `gh-pages` branch |

---

## Deployment and `homepage`

`package.json` includes:

```json
"homepage": "https://AdrianSalmonInteractives.github.io/fractal-explorer"
```

CRA injects this into **`PUBLIC_URL`** for asset paths. For **Content Interactives** hosting, set **`homepage`** to match the real GitHub Pages URL (e.g. `https://content-interactives.github.io/fractal-explorer`) before production builds, or relative paths may 404 under a different org/repo name.

---

## Performance / limits

- **Sierpinski / Koch:** depth 7 implies large primitive counts; acceptable on desktop browsers; very slow devices may stutter.
- **Dragon:** iteration 12 produces **2^12** segments in the turn model; polyline draw is still linear in segment count.

---

## Maintenance notes

- Prefer **`useRef` + canvas** instead of `document.getElementById` if the component tree grows (avoids ID collisions, improves React idioms).
- Surface **`info` and complexity strings** in the UI or remove unused state to reduce confusion.
- **`setInfo` inside draw functions** triggers a React state update on every paint → **extra re-renders**; consider deriving stats without `setState` in the effect or memoizing.

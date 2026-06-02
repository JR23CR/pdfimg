# Design: Banco de Activos + Comentarios por Foto + Portada Mejorada

## Technical Approach

Extend the single IIFE in `index.html` with three independent modules (asset-bank, photo-comments, enhanced-pdf-cover) organized via comment-section headers. Add ~350 lines to the existing file. Zero new files — single-file constraint preserved. All new code follows existing ES5 conventions (`var`, function expressions, callbacks).

## Architecture Decisions

| Decision | Choice | Alternatives | Rationale |
|----------|--------|-------------|-----------|
| Code organization | Comment-section modules within IIFE | Separate files, classes | Single-file constraint. Existing IIFE pattern. Section headers are the lightest coupling. |
| Asset persistence | localStorage key `pdfimg_assets` | IndexedDB | Only ~10KB of text data. localStorage is synchronous, simpler, sufficient. |
| Photo comments | In-memory only (`photo.comment`) | localStorage | Photos already in memory. Comments are session-scoped. Avoiding quota issues. |
| Asset selection | Single `selectedAssetId` var | Wrapped object | Matches existing `var` pattern. One asset = one id. |
| Pre-seed data | Separate `activos.json` file fetched on init | Inline JS const | User wants to edit the JSON directly. Fetch via XMLHttpRequest on first load when localStorage is empty. |
| Asset form | Modal overlay | Inline panel | Modal avoids layout shifts. Follows existing pattern (no framework). |
| PDF page break | Dynamic height calc, addPage when overflow | Fixed slots | Comments vary in length. Must measure `splitTextToSize` output and decide per page. |

## Data Flow

```
localStorage(pdfimg_assets)
        │
    [loadAssets()]             [loadImageFile()]
        │                            │
  assets[] ───── Asset Panel ──→ selectedAssetId
                                   photos[] ← photo.comment
        │                              │
    [generatePDF()]                    │
        │                              │
    ┌───┴──────────────────────────────┘
    │
    ├─ has asset? → Cover with asset table
    ├─ no asset?  → Existing cover (logo + title + count)
    └─ per photo page:
        ├─ layout 1-2 AND photo.comment? → render comment below
        └─ layout 3+ OR no comment      → current behavior
```

## Data Model

```js
// Asset (localStorage key: "pdfimg_assets")
{ id: "uuid", correlativo: "...", tipo: "maquinaria|equipo|vehículos",
  codigo: "...", descripcion: "...", no_serie: "...", no_modelo: "...",
  marca: "...", ubicacion: "...", placa: "...", chasis: "...",
  motor: "...", estado: "...", observaciones: "..." }

// Photo (extended — in memory only)
{ dataUrl: "base64...", w: 1920, h: 1080, comment: "" }
```

## UI Architecture

### Component Hierarchy (within `index.html`)

```
Header
└─ .container
   ├─ .drop-zone + .logo-indicator          ← unchanged
   ├─ .controls                             ← + "Banco Activos" toggle btn
   │   └─ + #assetToggleBtn                 ← new
   ├─ .asset-panel (collapsible)            ← new
   │   ├─ input#assetSearch                 ← new (filtro)
   │   ├─ table#assetTable                  ← new
   │   ├─ button#newAssetBtn                ← new
   │   └─ button#importAssetsBtn            ← new
   ├─ .grid#photoGrid                        ← unchanged, but cards get click handler
   └─ .toast#toast                          ← unchanged (reused for all toasts)

<!-- Modals (hidden by default) -->
div#assetFormModal                          ← new (overlay)
div#commentModal                            ← new (overlay)
```

### Interaction Flows

**Asset Bank Flow:**
1. User clicks "Banco Activos" toggle → `.asset-panel` slides open/close
2. Panel loads from `assets[]` (in-memory copy of localStorage)
3. Search input filters rows by código/descripción (keyup, real-time)
4. "Seleccionar" button → sets `selectedAssetId`, highlights row, updates UI indicator
5. "Editar" → opens asset form modal with fields pre-filled → save updates localStorage + assets[]
6. "Eliminar" → confirm dialog → splice from localStorage + assets[]. If it was selectedAssetId, clear it.
7. "Nuevo Activo" → opens asset form modal empty → save pushes to localStorage + assets[]
8. "Importar Lista" → inserts pre-seed data if not already present

**Photo Comment Flow:**
1. User clicks photo card in grid → `commentModal` opens with photo preview + textarea
2. If `photo.comment` exists, pre-fill textarea
3. "Guardar" → sets `photo.comment = textarea.value`, closes modal, re-renders grid (comment badge)
4. "Cancelar" / click outside → discard, close modal

**Enhanced Cover Flow:**
1. `generateBtn` handler checks `selectedAssetId`
2. If set → render cover with asset data table (logo + title + 13-row table)
3. If not set → existing cover (logo + title + photo count)
4. If portada toggle off → skip cover entirely (existing behavior)

## Code Organization

All within the existing IIFE, organized by clear comment-section headers:

```
┌─────────────────────────────────────────┐
│  (function() { 'use strict';            │
│                                          │
│  // ── DATA ──                          │
│  var photos = [];  // + comment field    │
│  var logo = null;                        │
│  var assets = [];         // NEW         │
│  var selectedAssetId = null; // NEW      │
│                                          │
│  // ── DOM REFS ──                      │
│  var dropZone = ...                      │
│  var assetToggleBtn = ...   // NEW       │
│  var assetPanel = ...       // NEW       │
│  var assetTableBody = ...   // NEW       │
│  var assetSearchInput = ... // NEW       │
│  var assetFormModal = ...   // NEW       │
│  var commentModal = ...     // NEW       │
│                                          │
│  // ── TOAST ── (unchanged)             │
│                                          │
│  // ── RENDER ── (modified)             │
│  // + comment badge in grid cards        │
│                                          │
│  // ── ASSET BANK ── (NEW)              │
│  function loadAssets() {}                │
│  function saveAssets() {}                │
│  function renderAssetTable() {}          │
│  function openAssetForm(assetOrNull) {}  │
│  function importDefaultAssets() {}       │
│                                          │
│  // ── PHOTO COMMENTS ── (NEW)          │
│  function openCommentModal(index) {}     │
│  function saveComment(index, text) {}    │
│                                          │
│  // ── PDF GENERATION ── (modified)     │
│  // + renderAssetTableOnCover()          │
│  // + renderPhotoComment()               │
│  // + measureCommentHeight()             │
│                                          │
│  // ── EVENTS ── (extended)             │
│  // + asset toggle, search, CRUD btns    │
│  // + photo click → comment modal        │
│                                          │
│  // ── INIT ──                           │
│  loadAssets();  // NEW                   │
│  render();                               │
│ })();                                    │
└─────────────────────────────────────────┘
```

### New Functions

| Function | Purpose |
|----------|---------|
| `loadAssets()` | Read `pdfimg_assets` from localStorage, parse JSON → `assets[]`. If empty/missing, call `importDefaultAssets()`. |
| `saveAssets()` | Serialize `assets[]` to JSON, write to `pdfimg_assets`. try/catch for QuotaExceededError → toast error. |
| `importDefaultAssets()` | Push the ~55 hardcoded assets into `assets[]` and call `saveAssets()`. |
| `renderAssetTable()` | Clear tbody, iterate `assets[]`, build rows with filter match. Show code, desc, type, marca, action buttons. |
| `openAssetForm(asset)` | Show modal overlay, fill 13 fields from asset or empty. On save: validate, update `assets[]`, `saveAssets()`, `renderAssetTable()`. |
| `openCommentModal(index)` | Show modal with photo preview + textarea. Pre-fill from `photos[index].comment`. |
| `saveComment(index, text)` | Set `photos[index].comment = text`, close modal, call `render()`. |
| `renderAssetTableOnCover(pdf, asset)` | Draw bordered 2-column table with 13 rows on the PDF cover page. |
| `renderPhotoComment(pdf, photo, x, y, maxW)` | Draw comment text below photo using `splitTextToSize`, 8pt, centered. Returns text height for page break calc. |

### Modified Functions

| Function | Change |
|----------|--------|
| `render()` | Add click handler on `.card img` → `openCommentModal(i)`. If `photo.comment`, show badge `.card .comment-badge`. |
| `generateBtn` handler | Add `if (selectedAssetId)` branch for cover page. After photo cell calculation, if layout ≤ 2 and `photo.comment`, subtract comment height from available photo area. |

## PDF Layout Spec

All measurements in mm.

### Cover Page — Asset Table

```
+--------------------------------------------------+
|  LOGO (centered, max 80×35mm)                    |
|                                                   |
|  ——— decorative line ———                          |
|  Título del álbum (26pt bold, centered)           |
|  ——— decorative line ———                          |
|                                                   |
|  FICHA TÉCNICA DEL ACTIVO (14pt bold, centered)   |
|                                                   |
|  +───────────────────┬───────────────────+        |
|  | Correlativo       | valor             |  ← row |
|  | Tipo              | valor             |    h=7  |
|  | Código            | valor             |         |
|  | Descripción       | valor             |         |
|  | No. Serie         | valor             |         |
|  | No. Modelo        | valor             |         |
|  | Marca             | valor             |         |
|  | Ubicación         | valor             |         |
|  | Placa             | valor             |         |
|  | Chasis            | valor             |         |
|  | Motor             | valor             |         |
|  | Estado            | valor             |         |
|  | Observaciones     | valor             |         |
|  +───────────────────┴───────────────────+        |
|                                                   |
|  Page footer (page X of Y)                        |
+--------------------------------------------------+
```

- **Table position**: Starts ~10mm below the title decorative line, or ~70mm from top
- **Column widths**: Left (campo) = 40% usableW, Right (valor) = 60% usableW
- **Row height**: 7mm
- **Borders**: `setDrawColor(200,200,200)`, `setLineWidth(0.2)`
- **Font**: Left = bold 8pt, Right = normal 8pt
- **Empty values**: render as "—" (em dash)
- **13 rows total height**: 13 × 7 = 91mm — fits comfortably on Letter (279mm) and A4 (297mm)

### Photo Page — Comment Below Photo

```
+--------------------------------------------------+
|  LOGO (top-right, max 50×18mm)                   |
|  Page X of Y                                      |
|                                                   |
|  ┌──────────────────────────────────────┐         |
|  │                                      │         |
|  │           PHOTO (aspect-ratio        │         |
|  │           preserved, centered)       │         |
|  │                                      │         |
|  └──────────────────────────────────────┘         |
|                                                   |
|  Comment text here (8pt normal, centered,         |
|  wrapped via splitTextToSize to usableW-2*PAD)    |
|                                                   |
+--------------------------------------------------+
```

- **Comment area**: Below photo, inside the inner border area (MARGIN + PAD)
- **Font**: 8pt, helvetica normal, `setTextColor(80,80,80)`
- **Max width**: `usableW - PAD * 2`
- **Line height**: ~3.5mm per line at 8pt
- **Vertical gap between photo and comment**: 3mm
- **Height calculation**: `commentHeight = splitTextToSize(...).length * 3.5 + 3`
- **Condition**: Comment ONLY rendered when `n <= 2` AND `photo.comment.length > 0`
- **Page break**: If photo natural height + comment text height exceeds `usableH - topSpace`, reduce photo height proportionally. If comment alone exceeds half the page, push to next page with photo only.

## Migration Plan

### Step 1: Add `comment` field to photos (safe, backward-compatible)
- `photos.push({ ...result, comment: '' })` instead of `photos.push(result)`
- Existing session photos (without .comment) → `photo.comment || ''` guard on read
- **No migration needed** — new property on new objects, defaults to undefined → falsy

### Step 2: Add asset bank (fully additive)
- `loadAssets()` on init — no effect until user interacts
- Pre-seed data only writes if localStorage key is absent
- All UI is behind collapsible panel — won't affect existing layout by default

### Step 3: Enhanced cover (conditional branch)
- Single `if (selectedAssetId)` branch in the cover section
- Without a selected asset → exact existing code path
- **Zero regression risk** — control flow splits at the top

### Step 4: Photo comments in PDF (conditional render)
- Check `layoutSel.value <= 2 && photo.comment` before rendering comment
- Layouts 3-4 completely unchanged
- Without comments → photos render at full available height (existing behavior)

### Rollback
Single git revert of the feature commit. No data migration needed — localStorage keys are namespaced (`pdfimg_assets`) and harmless if orphaned.

## Testing Strategy

| Layer | What | How |
|-------|------|-----|
| Manual | Asset CRUD cycle | Open panel → create → edit → delete → verify persistence across reload |
| Manual | Comment modal | Click photo → write comment → close → reopen → verify persisted in session |
| Manual | PDF output (with asset) | Select asset → generate → verify cover table + photo comments |
| Manual | PDF output (without asset) | No asset → generate → verify output identical to current |
| Manual | Filter/search | Type in search → verify table filters in real-time |
| Manual | Error handling | Fill localStorage to quota → attempt save → verify toast error |
| Manual | Responsive | Test panel collapse, modal layout on mobile viewport |

## Open Questions (Resolved)

- ✅ Pre-seed data → **Separate `activos.json`** fetched via XHR on init when localStorage is empty. User edits the JSON directly.
- ✅ Asset form → **2-column desktop, 1-column mobile**.
- ✅ Auto-seed → **Auto on empty localStorage** (no "Importar Lista" button needed).

## Additional Architecture Updates

- `importDefaultAssets()` now fetches `activos.json` via `XMLHttpRequest` instead of using an inline const.
- New file: `activos.json` in project root with the full asset list.
- `loadAssets()`: if localStorage key is missing or empty, fetch `activos.json`, parse, write to localStorage, then load into `assets[]`.

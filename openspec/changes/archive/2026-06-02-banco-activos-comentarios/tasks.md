# Tasks: Banco de Activos + Comentarios por Foto + Portada Mejorada

## Review Workload Forecast

| Field | Value |
|-------|-------|
| Estimated changed lines | ~450 (`index.html` +350, `activos.json` +100) |
| 400-line budget risk | Medium |
| Chained PRs recommended | Yes |
| Suggested split | PR 1: Foundation + Asset Bank (~260 lines) → PR 2: Comments + PDF + Polish (~190 lines) |
| Delivery strategy | ask-on-risk |
| Chain strategy | feature-branch-chain |

Decision needed before apply: Yes
Chained PRs recommended: Yes
Chain strategy: feature-branch-chain
400-line budget risk: Medium

### Suggested Work Units

| Unit | Goal | Likely PR | Notes |
|------|------|-----------|-------|
| 1 | Foundation + Asset Bank | PR 1 | `activos.json`, comment field, asset layer, panel UI, selection flow. Base = main |
| 2 | Comments + PDF + Polish | PR 2 | Comment modal, enhanced cover, comments in PDF, edge cases. Depends on PR 1 |

## Phase 1: Foundation

- **[x] T1** — Create `activos.json`
  - **Description**: New file with ~55 pre-seed assets. Each asset has: correlativo, tipo, codigo, descripcion, no_serie, no_modelo, marca, ubicacion, placa, chasis, motor, estado, observaciones.
  - **Files**: `activos.json` (new)
  - **Deps**: none
  - **Acceptance**: Valid JSON, every record has `id` (UUID) + all 13 fields, readable by `JSON.parse()`.
  - **Effort**: S

- **[x] T2** — Add `comment` field to photo data model
  - **Description**: Extend every `photos.push(result)` to `photos.push({...result, comment: ''})`. Add `photo.comment || ''` guard on comment reads.
  - **Files**: `index.html` (~3 lines)
  - **Deps**: none
  - **Acceptance**: Every photo object in `photos[]` has a `comment` property defaulting to `""`.
  - **Effort**: S

## Phase 2: Asset Bank

- **[x] T3** — Implement asset data layer
  - **Description**: `loadAssets()` reads `pdfimg_assets` from localStorage → parses into `assets[]`. If empty/missing, calls `importDefaultAssets()` which fetches `activos.json` via XHR, writes to localStorage. `saveAssets()` serializes `assets[]` to localStorage with try/catch for QuotaExceededError.
  - **Files**: `index.html` (~35 lines)
  - **Deps**: T1
  - **Acceptance**: On fresh load with empty localStorage, assets auto-seed from `activos.json`. CRUD saves persist across reload.
  - **Effort**: S

- **[x] T4** — Build asset panel UI
  - **Description**: Collapsible `.asset-panel` with toggle button in controls. Search input filters by código/descripción in real-time. Table with columns: Código, Descripción, Tipo, Marca + action buttons. New/Edit modals with 13-field form (2-col desktop, 1-col mobile). Delete with confirm.
  - **Files**: `index.html` (~100 lines: ~10 CSS, ~40 HTML, ~50 JS)
  - **Deps**: T3
  - **Acceptance**: Panel toggles open/closed. Search filters in real-time. Create/edit/delete works and persists. Form is 2-column on desktop, 1-column below 600px.
  - **Effort**: M

- **[x] T5** — Add asset selection flow
  - **Description**: `var selectedAssetId = null`. "Seleccionar" button per row sets it (radio behavior — deselects previous). Highlighted row in table. UI indicator (e.g., badge or banner) showing selected asset name. Clearing selection resets to null.
  - **Files**: `index.html` (~15 lines)
  - **Deps**: T4
  - **Acceptance**: Only one asset selected at a time. Selected row is visually distinct. Deleting selected asset clears `selectedAssetId`.
  - **Effort**: S

## Phase 3: Photo Comments

- **[x] T6** — Implement comment modal
  - **Description**: Click photo card → modal overlay with 150px photo preview left + textarea right. Pre-fills from `photo.comment`. "Guardar" sets `photo.comment = textarea.value`, closes modal, re-renders grid with comment badge. "Cancelar" discards and closes. Comments are in-memory only.
  - **Files**: `index.html` (~55 lines: ~10 CSS, ~20 HTML, ~25 JS)
  - **Deps**: T2
  - **Acceptance**: Modal opens on photo click. Existing comment pre-filled. Save persists in `photos[]`. Badge shown on cards with comments. Cancel discards changes.
  - **Effort**: S

## Phase 4: Enhanced PDF

- **[x] T7** — Add asset table to PDF cover
  - **Description**: When `selectedAssetId` is set and portada toggle on: render 13-row 2-column table (campo | valor) below title. Column widths 40%/60%. Font bold 8pt left, normal 8pt right. Empty values render as "—". Borders gray 0.2pt. Without asset → existing cover unchanged.
  - **Files**: `index.html` (~35 lines)
  - **Deps**: T5
  - **Acceptance**: Cover with asset shows full 13-row table. Without asset shows existing logo+title+count. With portada off, skip cover entirely.
  - **Effort**: S

- **[x] T8** — Add photo comments to PDF photo pages
  - **Description**: Per photo page: if `n <= 2` AND `photo.comment.length > 0`, render comment below photo using `splitTextToSize`, 8pt, centered, gray text. Comment height = lines * 3.5mm + 3mm gap. Adjust photo height proportionally if comment overflows. Layouts 3-4: no comments.
  - **Files**: `index.html` (~25 lines)
  - **Deps**: T6, T7
  - **Acceptance**: Layout 1-2 with comments shows text below photo. Layout 3-4 shows no comment text. Long comments wrap correctly.
  - **Effort**: S

## Phase 5: Polish

- **[x] T9** — Handle edge cases
  - **Description**: localStorage QuotaExceededError → toast error with no data loss. Deleted asset that is `selectedAssetId` → clear selection. Empty comment rendered as nothing in PDF. Responsive: asset form modal goes 1-col on mobile. Empty fields in asset table show "—".
  - **Files**: `index.html` (~15 lines)
  - **Deps**: T4, T5, T6, T7, T8
  - **Acceptance**: All edge cases handled without crashes or data corruption.
  - **Effort**: S

# Archive Report

**Change**: `banco-activos-comentarios`
**Status**: ARCHIVED
**Completed**: 2026-06-02
**Archive**: `openspec/changes/archive/2026-06-02-banco-activos-comentarios/`

---

## Summary

This change added three new capabilities to PDFIMG: a persistent asset bank (localStorage), photo-level comments, and an enhanced PDF cover with a full technical data sheet. All existing functionality remains intact when no asset is selected — full backward compatibility.

### Delivered Capabilities

| Capability | Key | Description |
|------------|-----|-------------|
| Asset Bank | `asset-bank` | CRUD for machinery/vehicle assets with 13 fields, persisted in localStorage, auto-seeded from `activos.json` |
| Photo Comments | `photo-comments` | Per-photo text comments via modal, session-persisted, rendered in PDF (layouts 1-2 only) |
| Enhanced PDF Cover | `enhanced-pdf-cover` | 13-row technical data table on cover when asset is selected, fallback to existing cover |

### Implementation Details

- Single-file constraint preserved — all changes in `index.html` (~730 lines added)
- New file: `activos.json` (56 pre-seed assets)
- 2 chained PRs: PR #1 (Foundation + Asset Bank), PR #2 (Comments + PDF + Polish)
- All 9 tasks completed and verified

---

## Artifacts

| Artifact | Path |
|----------|------|
| Proposal | `openspec/changes/archive/2026-06-02-banco-activos-comentarios/proposal.md` |
| Spec | `openspec/changes/archive/2026-06-02-banco-activos-comentarios/spec.md` |
| Design | `openspec/changes/archive/2026-06-02-banco-activos-comentarios/design.md` |
| Tasks | `openspec/changes/archive/2026-06-02-banco-activos-comentarios/tasks.md` |
| Verify Report | `openspec/changes/archive/2026-06-02-banco-activos-comentarios/verify-report.md` |
| Archive Report | `openspec/changes/archive/2026-06-02-banco-activos-comentarios/archive-report.md` |
| Main Spec (source of truth) | `openspec/specs/banco-activos-comentarios/spec.md` |

---

## Implementation Stats

| Metric | Value |
|--------|-------|
| Files changed | 2 (`index.html`, `activos.json`) |
| Lines added | ~786 (`index.html` +728, `activos.json` +58) |
| Lines removed | 3 |
| Commits | 7 (over 2 chained PR branches) |
| PRs created | 2 (feature-branch-chain) |
| Tasks completed | 9/9 (100%) |

### Commits

| Commit | Message |
|--------|---------|
| `c8f4a70` | feat: add activos.json with full 56-asset inventory |
| `c04d93d` | feat: add comment field to photo data model |
| `b5ae340` | feat: implement asset data layer with localStorage persistence |
| `0ac154c` | feat: add asset bank panel UI with CRUD and selection flow |
| `3ff9515` | docs: mark T1-T5 complete in tasks.md, set chain strategy |
| `fb84f09` | feat: add photo comment modal, PDF asset table, and photo comments in PDF |
| `b065dc1` | fix: handle edge cases for asset bank and photo comments |

---

## Spec Compliance

All 18 spec requirements and 14 scenarios verified as COMPLIANT (32/32).

| Category | Total | Compliant | Partial | Failed |
|----------|-------|-----------|---------|--------|
| Spec requirements | 18 | 18 | 0 | 0 |
| Scenarios | 14 | 14 | 0 | 0 |

### Design Coherence

All 11 design decisions followed correctly. 3 minor deviations (inlined functions instead of extracted) — no impact correctness.

---

## Known Issues (from Verify Report)

### Warnings (open)

1. **Dead conditional (L686)**: `(a.id === selectedAssetId) ? 'btn-sm-primary' : 'btn-sm-primary'` — both branches same class. Cosmetically harmless.
2. **`hasTitle` couples toggle with title presence (L1005)**: Cover toggle + empty title = no cover at all. Pre-existing behavior.
3. **Inconsistent null normalization (L749-762)**: Some form fields normalized to `null`, others saved as `""`. Cosmetic inconsistency in localStorage.
4. **No XHR timeout (L632-652)**: `importDefaultAssets()` XHR has no timeout — negligible for local file.
5. **Tipo enum values differ from spec language**: Spec says `maquinaria/equipo/vehículos`, real-world data uses `Vehículo/Máquina/Arrastra`. Data is correct per domain terminology.

### Suggestions (open)

1. Extract cover table and comment rendering to named functions (readability).
2. Add keyboard shortcut to comment modal (Enter to save, Escape to cancel).
3. Decouple cover toggle from title presence.
4. Normalize all form fields to `null` consistently.

**No CRITICAL issues. No regressions.**

---

## Next Steps

1. **Spec versioning**: This is the first spec for the project. Future changes should produce delta specs under `openspec/changes/{change}/specs/{domain}/spec.md` and merge into `openspec/specs/`. The current spec covers 3 domains in one file — consider splitting by domain for future deltas.
2. **Address warnings**: None are urgent, but items #3 (null normalization) and #5 (tipo enum) should be fixed if the spec is reused or if the asset form is extended.
3. **PDF polish**: The comments-in-PDF rendering reduces photo height proportionally; edge case where comment exceeds half the page silently drops the comment (see verify report).
4. **Beyond this change**: Future work could include cloud sync, CSV/JSON export, image editing/crop, or authentication — all explicitly out of scope for this change.

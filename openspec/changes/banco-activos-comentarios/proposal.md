# Proposal: Banco de Activos + Comentarios por Foto + Portada Mejorada

## Intent

El usuario necesita documentar activos de maquinaria/vehículos con fotos. Hoy la app solo permite fotos anónimas sin metadatos. Este cambio agrega un banco de activos persistente (localStorage), comentarios por foto, y una portada de PDF con ficha técnica completa.

## Scope

### In Scope
1. **Banco de Activos** — CRUD completo con 13 campos (correlativo, tipo, código, descripción, serie, modelo, marca, ubicación, placa, chasis, motor, estado, observaciones) persistido en localStorage
2. **Comentarios por Foto** — Modal al hacer clic en una foto para escribir/editar su descripción; el comentario viaja con la foto al PDF
3. **Portada de PDF Mejorada** — Si hay un activo seleccionado, la portada muestra logo + título + tabla con todos los datos del activo
4. **Páginas de Fotos con Comentario** — Cada foto ocupa la página con su comentario debajo (ajustable según layout)
5. Compatibilidad total hacia atrás: todo lo existente sigue funcionando sin activo seleccionado

### Out of Scope
- Autenticación, usuarios, roles
- Sincronización cloud o exportación CSV/JSON de activos
- Edición/crop de imágenes
- Impresión de comentarios en layouts de 3+ fotos por página (solo 1-2 fotos por página con comentarios)

## Capabilities

### New Capabilities
- `asset-bank`: CRUD de activos con localStorage, selección y pre-populado de PDF
- `photo-comments`: Comentario de texto por foto, modal de edición, renderizado en PDF
- `enhanced-pdf-cover`: Portada con ficha técnica del activo seleccionado

### Modified Capabilities
None — primera spec del proyecto.

## Approach

**Data model**: Extender `photos[]` a `{ dataUrl, w, h, comment }`. Nuevo `assets[]` en localStorage bajo key `pdfimg_assets`.

**UI**: Panel colapsable "Banco de Activos" debajo del grid. Tabla con búsqueda/filtro. Modal para crear/editar activo. Selector de activo activa la portada mejorada. Comentarios: clic en foto abre modal con textarea.

**PDF**: Si hay activo seleccionado, portada = logo + título + tabla de datos del activo. Páginas de foto: layout respeta aspect ratio, comentario abajo en fuente pequeña. Sin activo = comportamiento actual intacto.

**Persistencia**: Assets se guardan en localStorage (metadata, no fotos). Fotos y comentarios en memoria de sesión únicamente — no se persisten fotos en localStorage por límite de cuota.

## Affected Areas

| Area | Impact | Description |
|------|--------|-------------|
| `index.html` | Modified | ~+350 líneas JS + HTML + CSS para banco, modal, comentarios, PDF extendido |

## Risks

| Risk | Likelihood | Mitigation |
|------|------------|------------|
| Quota localStorage excedida con ~55 activos | Low | Solo texto metadata (~10KB), fotos quedan en memoria |
| Regresión en PDF existente sin activo | Low | Branch condicional: sin activo = código original intacto |
| UX saturada en pantalla chica | Med | Panel colapsable, modal overlay, responsive existing |

## Rollback Plan

Descartar la rama. `index.html` original está en `main`. Un commit, un revert.

## Dependencies

- jsPDF 2.5.1 (ya cargado vía CDN) — sin cambios

## Success Criteria

- [ ] Cargar ~55 activos desde localStorage, verlos en tabla, filtrar por código/descripción
- [ ] Seleccionar un activo → portada PDF con todos sus datos
- [ ] Hacer clic en foto → modal → escribir comentario → se guarda en memoria
- [ ] PDF generado con activo: portada + fotos con comentario debajo
- [ ] PDF generado sin activo: idéntico al comportamiento actual

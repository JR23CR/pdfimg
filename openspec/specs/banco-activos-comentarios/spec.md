# Especificaciones: Banco de Activos + Comentarios por Foto + Portada Mejorada

**Change**: `banco-activos-comentarios`
**Tipo**: Nuevas capacidades (3 dominios) — no hay specs previas.

---

## Overview

Este cambio agrega tres capacidades nuevas a PDFIMG: un banco de activos persistente en localStorage, comentarios por foto, y una portada de PDF mejorada con ficha técnica del activo seleccionado. Sin activo seleccionado, el comportamiento existente se mantiene intacto.

---

## 1. Asset Bank (asset-bank)

### Requerimientos

| # | Requerimiento | RFC 2119 |
|---|---------------|----------|
| AB-1 | Persistir activos en localStorage bajo clave `pdfimg_assets` | MUST |
| AB-2 | Cada activo DEBE tener: correlativo, tipo (maquinaria/equipo/vehículos), código, descripción, no.serie, no.modelo, marca, ubicación, placa, chasis, motor, estado, observaciones | MUST |
| AB-3 | CRUD completo: crear, editar, eliminar, seleccionar activo activo | MUST |
| AB-4 | Importar lista inicial (~55 activos) si localStorage está vacío al cargar | MUST |
| AB-5 | Tabla de activos con búsqueda/filtro por código y descripción | MUST |
| AB-6 | Solo un activo seleccionable a la vez (radio behavior) | MUST |

### Escenarios

#### AB-H1: Importar lista inicial
- GIVEN localStorage vacío y app recién cargada
- WHEN el sistema detecta que no hay activos
- THEN importa la lista predefinida de ~55 activos y los persiste en `pdfimg_assets`

#### AB-H2: CRUD — crear activo manual
- GIVEN el panel de banco de activos está abierto
- WHEN el usuario completa el formulario de nuevo activo y guarda
- THEN el activo se agrega a localStorage y aparece en la tabla

#### AB-H3: CRUD — editar activo
- GIVEN un activo existe en la tabla
- WHEN el usuario hace clic en "Editar", modifica campos y guarda
- THEN los cambios se persisten en localStorage

#### AB-H4: CRUD — eliminar activo
- GIVEN un activo existe en la tabla
- WHEN el usuario hace clic en "Eliminar" y confirma
- THEN el activo se elimina de localStorage y de la tabla

#### AB-H5: Seleccionar activo activo
- GIVEN hay activos en el banco
- WHEN el usuario hace clic en "Seleccionar" sobre un activo
- THEN ese activo queda como seleccionado (highlight en tabla)
- AND si había otro seleccionado, se deselecciona

#### AB-E1: Eliminar activo seleccionado
- GIVEN el activo actualmente seleccionado es eliminado
- THEN la selección activa se limpia a null

#### AB-E2: Quota excedida en localStorage
- GIVEN localStorage está cerca del límite
- WHEN el usuario intenta guardar/importar activos
- THEN el sistema muestra toast de error y no pierde datos existentes

---

## 2. Photo Comments (photo-comments)

### Requerimientos

| # | Requerimiento | RFC 2119 |
|---|---------------|----------|
| PC-1 | Modelo `photo` DEBE extender con campo `comment` (string, default "") | MUST |
| PC-2 | Clic en foto del grid DEBE abrir modal con textarea para escribir/editar comentario | MUST |
| PC-3 | Comentario DEBE persistir en memoria de sesión (no localStorage) junto a la foto | MUST |
| PC-4 | Modal DEBE mostrar preview de la foto + textarea lado a lado | MUST |
| PC-5 | Comentario DEBE renderizarse en PDF debajo de su foto (1-2 fotos/página únicamente) | MUST |
| PC-6 | Grid DEBE mostrar indicador visual si una foto tiene comentario | SHOULD |

### Escenarios

#### PC-H1: Agregar comentario
- GIVEN hay fotos en el grid
- WHEN el usuario hace clic en una foto sin comentario
- THEN se abre un modal con la foto y un textarea vacío
- WHEN el usuario escribe texto y confirma
- THEN `photo.comment` se actualiza en memoria y el grid muestra indicador

#### PC-H2: Editar comentario existente
- GIVEN una foto ya tiene comentario
- WHEN el usuario hace clic en la foto
- THEN el modal se abre con el comentario precargado en el textarea

#### PC-H3: Cerrar modal sin guardar
- GIVEN el modal de comentario está abierto
- WHEN el usuario hace clic en Cancelar o fuera del modal
- THEN el comentario NO se modifica (se descartan cambios)

#### PC-E1: Comentario vacío
- GIVEN el modal de comentario está abierto
- WHEN el usuario guarda con el textarea vacío
- THEN `photo.comment` se setea a "" (string vacío)
- AND el PDF muestra la foto sin texto debajo

#### PC-E2: Foto sin comentario en layouts 3+
- GIVEN el layout está configurado a 3 o 4 fotos por página
- WHEN se genera el PDF
- THEN ninguna foto muestra comentario debajo (layout no modificado)

---

## 3. Enhanced PDF Cover (enhanced-pdf-cover)

### Requerimientos

| # | Requerimiento | RFC 2119 |
|---|---------------|----------|
| EP-1 | Portada con activo seleccionado DEBE mostrar: logo + título + tabla con TODOS los campos del activo | MUST |
| EP-2 | Sin activo seleccionado → portada DEBE usar comportamiento existente (solo logo + título + contador) | MUST |
| EP-3 | Si portada desactivada (toggle) → NO generarla, empezar directo con fotos (con o sin activo) | MUST |
| EP-4 | Páginas con 1-2 fotos: comentario DEBE aparecer debajo de la foto, centrado, fuente 8pt | MUST |
| EP-5 | Páginas con 3+ fotos: comentario NO DEBE mostrarse | MUST |
| EP-6 | Tabla de ficha técnica DEBE tener formato 2-columnas (campo | valor) con bordes finos | MUST |

### Escenarios

#### EP-H1: PDF con activo — portada con ficha técnica
- GIVEN el usuario seleccionó un activo en el banco
- WHEN genera PDF con portada activada
- THEN página 1: logo centrado arriba, título del álbum, tabla con 13 filas (campo + valor)
- AND páginas siguientes: fotos con comentarios debajo (si aplica según layout)

#### EP-H2: PDF sin activo — comportamiento legacy
- GIVEN no hay activo seleccionado
- WHEN genera PDF con portada activada
- THEN portada = logo + título + "N fotos" (idéntico al código actual)

#### EP-H3: PDF con activo y portada desactivada
- GIVEN hay activo seleccionado pero portada desactivada
- WHEN genera PDF
- THEN PDF empieza directo con fotos, sin portada

#### EP-E1: Comentario largo en layout 1 foto
- GIVEN el comentario de una foto excede 200 caracteres
- WHEN se genera PDF con 1 foto por página
- THEN jsPDF `splitTextToSize` DEBE dividir el texto en líneas
- AND el texto DEBE caber dentro del ancho usable sin cortarse

#### EP-E2: Activo sin algunos campos llenos
- GIVEN el activo seleccionado tiene campos vacíos (ej. sin placa ni chasis)
- WHEN se genera la portada
- THEN la tabla muestra el campo con valor "-" o vacío
- AND el diseño de la tabla no se rompe por valores ausentes

---

## Data Model

### Asset (localStorage key: `pdfimg_assets`)
```json
{
  "id": "string (UUID)",
  "correlativo": "string",
  "tipo": "maquinaria | equipo | vehículos",
  "codigo": "string",
  "descripcion": "string",
  "no_serie": "string",
  "no_modelo": "string",
  "marca": "string",
  "ubicacion": "string",
  "placa": "string",
  "chasis": "string",
  "motor": "string",
  "estado": "string",
  "observaciones": "string"
}
```

### Photo (extendido — en memoria)
```json
{
  "dataUrl": "string (base64)",
  "w": "number",
  "h": "number",
  "comment": "string (nuevo, default '')"
}
```

---

## UI Specifications

**Banco de Activos**: Panel colapsable debajo del grid con botón toggle "Banco de Activos". Tabla con columnas Código, Descripción, Tipo, Marca. Input de búsqueda en tiempo real. Botones por fila: Seleccionar (radio), Editar, Eliminar. Botones globales: Nuevo Activo, Importar Lista. Modal de formulario con 13 campos en two-column layout (desktop) o one-column (mobile).

**Modal de Comentario**: Overlay al hacer clic en foto. Layout: preview foto 150px izquierda + textarea derecha. Botones Guardar/Cancelar. Indicador visual en el grid (ícono 💬 o badge) si la foto tiene comentario.

**Portada Mejorada**: Logo centrado (max 80×35mm). Título del álbum. Tabla "FICHA TÉCNICA DEL ACTIVO" con 13 filas, 2 columnas (campo negrita | valor normal), bordes grises finos, ancho usable completo.

---

## Out of Scope

- Autenticación, usuarios, roles
- Sincronización cloud, exportación CSV/JSON
- Edición o crop de imágenes
- Comentarios en layouts de 3+ fotos por página
- Persistencia de fotos en localStorage

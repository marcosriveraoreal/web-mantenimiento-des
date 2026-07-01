# Bocetos Mantenimiento — Atalaya Mining

Prototipos HTML/CSS/JS estáticos para el sistema de gestión de solicitudes de personal de mantenimiento. Sin framework, sin build step, sin servidor — todo funciona abriéndolo directamente en el navegador.

## Arquitectura

Dos aplicaciones independientes que comparten el mismo `localStorage`:

| App | Archivo(s) | Acceso |
|-----|-----------|--------|
| **Portal externo** (proveedores) | `login.html` → `portal.html` → `solicitudes.html` | Suministradores |
| **App interna** (staff Atalaya) | `solicitud_personal.html` ↔ `tabla_solicitudes.html` | Equipo interno |

## Persistencia

| Clave | Contenido |
|-------|-----------|
| `localStorage["sdi_solicitudes"]` | Array de solicitudes (shared entre las dos apps) |
| `localStorage["sdi_edit_id"]` | ID de la solicitud que se va a editar (string) — lo escribe `tabla_solicitudes.html` y lo consume y borra `solicitud_personal.html` |
| `sessionStorage["atalaya_session"]` | Sesión del portal externo: `{ username, company }` |

## Modelo de datos

```js
// Solicitud
{
  id: Number,                    // Date.now()
  numero_solicitud: String,      // "0001"
  createdAt: String,             // ISO date
  observaciones_generales: String,
  filas: [
    {
      fecha: String,             // "YYYY-MM-DD"
      turno: String,
      suministrador: String,     // debe ser uno de SUMINISTRADORES
      categoria: String,
      zona: String,
      solicitado: Number,
      contrata: Number,          // lo rellena el proveedor desde el portal
      real: Number,              // lo rellena el staff interno (solo si aprobado_interno)
      observaciones: String,
      adjunto: String|null,
      estado: String,            // ver máquina de estados más abajo
      // si la línea está rechazada internamente:
      motivo_rechazo: String,
      rechazadoAt: String,       // ISO date
      // si la línea está cancelada:
      cancelada: true,
      motivo_cancelacion: String,
      canceladaAt: String,       // ISO date
    }
  ]
}
```

## Máquina de estados de fila (`estado`)

```
no_solicitado
    │ (staff envía la línea)
    ▼
solicitado
    │ (proveedor confirma desde portal externo)
    ▼
confirmado_externo
    │ (staff aprueba)           │ (staff rechaza con motivo)
    ▼                           ▼
aprobado_interno         rechazado_interno
    │                           │
    │ (rechazo es definitivo)   │ (proveedor no puede re-confirmar)
    │
    │ (staff guarda campo Real)
    ▼
real_aceptado

En cualquier estado (excepto cancelada):
    → cancelada  (staff cancela la línea con motivo)
```

La función `getEstado(f)` en `tabla_solicitudes.html` y `solicitudes.html` deriva el estado:
- Si `f.cancelada` → `'cancelada'`
- Si `f.estado` existe → usa ese valor
- Si `f.real > 0` → `'real_aceptado'` (legacy)
- Si `f.contrata > 0` → `'confirmado_externo'` (legacy)
- En otro caso → `'solicitado'`

## Flujo principal

1. **Staff** crea solicitud en `solicitud_personal.html` → se guarda en `sdi_solicitudes`
2. **Staff** envía líneas a proveedores desde `tabla_solicitudes.html` (botón "Enviar a proveedores" o por línea)
3. **Proveedor** entra al portal (`login.html`), ve sus solicitudes en `solicitudes.html` y rellena el campo **Confirmado** (nº personas disponibles)
4. **Staff** revisa en `tabla_solicitudes.html`: aprueba (`aprobado_interno`) o rechaza con motivo (`rechazado_interno`) cada línea confirmada
   - Si rechaza, el rechazo es **definitivo** — el proveedor lo ve en lectura, no puede re-confirmar
5. **Staff** rellena el campo **Real** solo en las líneas `aprobado_interno`

## Drawer de `tabla_solicitudes.html`

El drawer se abre en dos modos:
- **Vista** (`openDrawer(id, 'view')`): solo lectura, sin botones de acción, sin inputs editables. Se activa con el icono ojo (👁) de la tabla.
- **Edición** (`openDrawer(id, 'edit')`): muestra acciones por línea (Aprobar/Rechazar/Cancelar/Enviar), inputs Real editables (solo en `aprobado_interno`), botón "Guardar Real". Se activa con el icono lápiz (✏) de la tabla.

## Listas de valores (deben coincidir en todos los archivos)

```js
const SUMINISTRADORES = ['Insersa', 'Royman', 'Mimese', 'Ventura'];
const TURNOS   = ['7:00 - 15:00','15:00 - 23:00','23:00 - 7:00','8:00 - 17:00','8:00 - 20:00','20:00 - 8:00'];
const CATEGORIAS = ['Oficial 1ª','Oficial 2ª','Peón','Categoría 1','Categoría 2','Categoría 3'];
const ZONAS    = ['Trituración','Molienda','Taller','Trabaux'];
```

Si se añade un proveedor nuevo hay que actualizar esta lista en `solicitud_personal.html`, `tabla_solicitudes.html` y `solicitudes.html` (portal).

## Usuarios del portal externo

Credenciales hardcoded en `login.html`. Cada usuario ve solo sus propias solicitudes.

| Usuario | Contraseña | Empresa |
|---------|-----------|---------|
| insersa | Insersa123 | Insersa |
| royman  | Royman123  | Royman  |
| mimese  | Mimese123  | Mimese  |
| ventura | Ventura123 | Ventura |

## Diseño

- **App interna**: sidebar verde oscuro (`--p: #194447`), fondo gris claro, sin framework
- **Portal externo**: topbar verde (`--primary: #194447`), fondo blanco/teal, diseño más limpio
- Variables CSS (design tokens) definidas en `:root` de cada archivo — no hay hoja de estilos compartida
- El isotipo SVG de Atalaya está en `isotipo.svg` e inline en los topbars

## Convenciones

- IDs de solicitud: `Date.now()` (número). Comparar siempre con `===` tras convertir con `+` el valor del atributo `data-id`
- El campo `suministrador` en las filas debe coincidir exactamente (case-sensitive) con `SUMINISTRADORES` — el drawer de `tabla_solicitudes.html` agrupa por ese campo
- Nada de módulos ES, nada de imports — todo en un único `<script>` al final del `<body>`
- El toast (`.toast`) tiene `pointer-events: none` siempre — es una notificación sin interactividad, y sin este CSS bloquearía clics en el footer del drawer al estar en posición fixed

## Notas técnicas

- La tabla principal **no** muestra columna de estado global — el estado se gestiona línea a línea en el drawer
- El campo **Real** solo es editable cuando la línea está en estado `aprobado_interno`
- El rechazo interno (`rechazado_interno`) es definitivo desde el portal externo: el proveedor lo ve en lectura sin opción de re-confirmar

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
      real: Number,              // lo rellena el staff interno
      observaciones: String,
      adjunto: String|null,
      // si la línea está rechazada:
      cancelada: true,
      motivo_cancelacion: String,
      canceladaAt: String,       // ISO date
    }
  ]
}
```

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

## Flujo principal

1. Staff crea solicitud en `solicitud_personal.html` → se guarda en `sdi_solicitudes`
2. Proveedor entra al portal (`login.html`), ve sus solicitudes en `solicitudes.html` y rellena los campos **Confirmado** y **Real**
3. Staff revisa en `tabla_solicitudes.html`: abre el drawer de detalle para ver líneas, puede editar, actualizar el campo **Real** interno, o rechazar líneas individuales con motivo

## Diseño

- **App interna**: sidebar verde oscuro (`--p: #194447`), fondo gris claro, sin framework
- **Portal externo**: topbar verde (`--primary: #194447`), fondo blanco/teal, diseño más limpio
- Variables CSS (design tokens) definidas en `:root` de cada archivo — no hay hoja de estilos compartida
- El isotipo SVG de Atalaya está en `isotipo.svg` e inline en los topbars

## Convenciones

- IDs de solicitud: `Date.now()` (número). Comparar siempre con `===` tras convertir con `+` el valor del atributo `data-id`
- El campo `suministrador` en las filas debe coincidir exactamente (case-sensitive) con `SUMINISTRADORES` — el drawer de `tabla_solicitudes.html` agrupa por ese campo
- Nada de módulos ES, nada de imports — todo en un único `<script>` al final del `<body>`
- Los estados del badge en la tabla: `pending` (0 confirmados), `partial` (parcialmente), `confirmed` (todos confirmados)

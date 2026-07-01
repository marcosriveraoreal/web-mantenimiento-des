# Bocetos Mantenimiento — Atalaya Mining

Prototipos HTML/CSS/JS estáticos para el sistema de gestión de solicitudes de personal y maquinaria auxiliar de mantenimiento. Sin framework, sin build step, sin servidor — todo funciona abriéndolo directamente en el navegador.

## Arquitectura

Dos módulos independientes que comparten el mismo navegador/localStorage:

| Módulo | Archivo(s) | Acceso |
|--------|-----------|--------|
| **Portal externo** (proveedores) | `login.html` → `portal.html` → `solicitudes.html` | Suministradores |
| **Solicitud personal** (staff) | `solicitud_personal.html` ↔ `tabla_solicitudes.html` | Equipo interno |
| **Maquinaria auxiliar** (staff) | `solicitud_generadores.html` ↔ `tabla_generadores.html` | Equipo interno |

## Persistencia

| Clave | Contenido |
|-------|-----------|
| `localStorage["sdi_solicitudes"]` | Array de solicitudes de personal (shared entre app interna y portal externo) |
| `localStorage["sdi_edit_id"]` | ID de la solicitud a editar — lo escribe `tabla_solicitudes.html` y lo consume y borra `solicitud_personal.html` |
| `localStorage["sdi_generadores"]` | Array de registros de maquinaria auxiliar |
| `sessionStorage["atalaya_session"]` | Sesión del portal externo: `{ username, company }` |

## Modelo de datos — Solicitud de personal

```js
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
      real: Number,              // lo rellena el staff (solo si aprobado_interno)
      observaciones: String,
      adjunto: String|null,
      estado: String,
      motivo_rechazo: String,
      rechazadoAt: String,
      cancelada: true,
      motivo_cancelacion: String,
      canceladaAt: String,
    }
  ]
}
```

## Modelo de datos — Maquinaria auxiliar

```js
{
  id: Number,                     // Date.now()
  createdAt: String,              // ISO date
  fecha: String,                  // "YYYY-MM-DD" — fecha del registro
  observaciones_generales: String,
  filas: [
    {
      empresa: String,            // uno de EMPRESAS
      zona: String,               // uno de ZONAS
      dia_solicitado: String,     // "YYYY-MM-DD"
      potencia_solicitada: Number, // kW (opcional)
      medios_propios: String,     // 'si'|'no'|'carga'|'descarga'|'carga_descarga'
      n_equipo_fabricante: String,
      n_equipo_atalaya: String,
      gasoil_entrada: Number,
      gasoil_salida: Number,      // se completa tras entrega
      ot_sc: String,
      telefono: String,
      extintor: String,           // 'si'|'no'
      picatierra: String,         // 'si'|'no'
      telefono_tecnico: String,
      dia_entrega: String,        // "YYYY-MM-DD" — se completa tras entrega
      dia_baja: String,           // "YYYY-MM-DD"
      dia_recogida: String,       // "YYYY-MM-DD" — cierra el ciclo
      estim_horas_dia: Number,
      potencia_entregada: Number, // kW — se completa tras entrega
    }
  ]
}
```

## Estados de línea — Maquinaria auxiliar (`lineStatus`)

```
pending   → sin dia_entrega
partial   → tiene dia_entrega pero no dia_recogida
completed → tiene dia_recogida
```

`globalStatus(rec)` agrega los estados de todas las filas:
- todas `completed` → `completed`
- alguna `partial` o `completed` → `partial`
- resto → `pending`

## Máquina de estados — Solicitud de personal (`estado`)

```
no_solicitado → solicitado → confirmado_externo → aprobado_interno → real_aceptado
                                               ↘ rechazado_interno (definitivo)
En cualquier estado → cancelada
```

`getEstado(f)` en `tabla_solicitudes.html` y `solicitudes.html`:
- `f.cancelada` → `'cancelada'`
- `f.estado` existe → usa ese valor
- `f.real > 0` → `'real_aceptado'` (legacy)
- `f.contrata > 0` → `'confirmado_externo'` (legacy)
- Resto → `'solicitado'`

## Drawer de `tabla_solicitudes.html`

- **Vista** (`openDrawer(id, 'view')`): solo lectura, sin inputs editables. Icono ojo (👁).
- **Edición** (`openDrawer(id, 'edit')`): acciones por línea, inputs Real editables (solo `aprobado_interno`), botón "Guardar Real". Icono lápiz (✏).

## Drawer de `tabla_generadores.html`

Ambos modos usan el mismo layout de cards por línea (una card por fila, 4 filas de grid):

- **Vista** (`openDrawer(id, 'view')`): cards con todos los campos deshabilitados (`disabled`). La fecha y observaciones del registro también aparecen deshabilitadas en la parte superior. El header de cada card muestra el punto de color de empresa + nombre.
- **Edición** (`openDrawer(id, 'edit')`): mismas cards con todos los campos editables — incluyendo `empresa` (select), `zona` (select), `dia_solicitado`, `potencia_solicitada`, `medios_propios`, `gasoil_entrada`, `telefono`, `ot_sc`, `extintor`, `picatierra`, `telefono_tecnico`, `estim_horas_dia`, `n_equipo_fabricante`, `n_equipo_atalaya`, `gasoil_salida`, `dia_entrega`, `dia_baja`, `dia_recogida`, `potencia_entregada`. También edita `fecha` y `observaciones_generales` del registro.

El guardado recoge todos los campos con `[data-gi][data-field]` + los campos de registro (`edit-fecha`, `edit-obs`).

## Formulario `solicitud_generadores.html`

Layout card-based por línea (igual que `solicitud_personal.html`):
- **rc-r1**: Empresa *, Zona *, Día solicitado *, Pot. solicitada (opt.)
- **rc-r2**: Medios propios, Nº eq. fabricante, Nº eq. Atalaya, Gasoil E, Telf. contacto
- **rc-r3**: OT/SC, Extintor, Picatierra, Gasoil S, Telf. técnico
- **rc-r4**: Día entrega, Día baja, Día recogida, Est. h/día, Pot. entregada

Live header tags en cada card: empresa, día solicitado (formateado), zona (badge oscuro).

## Listas de valores

### Solicitud de personal (deben coincidir en `solicitud_personal.html`, `tabla_solicitudes.html`, `solicitudes.html`)
```js
const SUMINISTRADORES = ['Insersa', 'Royman', 'Mimese', 'Ventura'];
const TURNOS   = ['7:00 - 15:00','15:00 - 23:00','23:00 - 7:00','8:00 - 17:00','8:00 - 20:00','20:00 - 8:00'];
const CATEGORIAS = ['Oficial 1ª','Oficial 2ª','Peón','Categoría 1','Categoría 2','Categoría 3'];
const ZONAS    = ['Trituración','Molienda','Taller','Trabaux'];
```

### Maquinaria auxiliar (deben coincidir en `solicitud_generadores.html` y `tabla_generadores.html`)
```js
const EMPRESAS = ['Royman','Mimese','Insersa','Ventura','Castrosur','Umaco','Nervion','SyL','CSD'];
const ZONAS    = ['Trituración','Molienda','Taller','Almacén','Cargadero'];
const EMP_COLORS = {
  'Insersa':'#009fe3','Royman':'#eb953f','Mimese':'#1BA777','Ventura':'#8b5cf6',
  'Castrosur':'#194447','Umaco':'#cd002b','Nervion':'#0891b2','SyL':'#e11d48','CSD':'#6b6e72',
};
```

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
- El isotipo SVG de Atalaya está inline en los sidebars/topbars de todos los archivos

## Convenciones

- IDs: `Date.now()` (número). Comparar siempre con `===` tras convertir con `+` el atributo `data-id`
- El campo `suministrador`/`empresa` debe coincidir exactamente (case-sensitive) con sus listas
- Nada de módulos ES, nada de imports — todo en un único `<script>` al final del `<body>`
- El toast (`.toast`) tiene `pointer-events: none` siempre — evita que bloquee clics en el footer del drawer
- `migrateRecords()` en `tabla_generadores.html` normaliza datos legacy al cargar

## Notas técnicas

- La tabla principal no muestra columna de estado global — el estado se gestiona línea a línea en el drawer
- El campo **Real** (personal) solo es editable cuando la línea está en `aprobado_interno`
- El rechazo interno es definitivo: el proveedor lo ve en lectura sin opción de re-confirmar
- Los campos de entrega de maquinaria (`dia_entrega`, `dia_recogida`, etc.) se pueden editar desde el formulario de creación o desde el drawer de la tabla

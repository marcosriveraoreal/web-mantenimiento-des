# Portal de Gestión de Personal de Mantenimiento — Atalaya Mining

Prototipo estático del sistema de gestión de solicitudes de personal de mantenimiento. Funciona directamente en el navegador sin servidor ni instalación.

## Cómo ejecutar

Abre cualquiera de los archivos `.html` directamente en el navegador (doble clic o arrastrar a Chrome/Edge). No requiere servidor web.

## Aplicaciones

### App interna (staff Atalaya)

| Archivo | Función |
|---------|---------|
| `solicitud_personal.html` | Crear y editar solicitudes de personal |
| `tabla_solicitudes.html` | Gestionar solicitudes: ver, aprobar/rechazar confirmaciones, registrar Real |

### Portal externo (proveedores)

| Archivo | Función |
|---------|---------|
| `login.html` | Acceso al portal (credenciales por empresa) |
| `portal.html` | Pantalla de bienvenida tras el login |
| `solicitudes.html` | Ver solicitudes asignadas y confirmar personal disponible |

## Flujo de trabajo

```
Staff (app interna)                    Proveedor (portal externo)
─────────────────────                  ──────────────────────────
1. Crea solicitud
2. Envía líneas al proveedor    ──▶    3. Confirma personal disponible
4. Aprueba o rechaza
   cada línea confirmada
5. Registra personal Real
   (solo líneas aprobadas)
```

### Estados de cada línea

| Estado | Descripción |
|--------|-------------|
| No enviado | Creada pero no enviada al proveedor |
| Enviado | Enviada al proveedor, esperando confirmación |
| Conf. ext. | Proveedor ha confirmado — pendiente revisión interna |
| Aprobado | Staff aprueba la confirmación — se puede rellenar Real |
| Rechazado | Staff rechaza la confirmación (definitivo) |
| Cancelada | Línea cancelada por el staff |

## Acceso al portal externo

| Usuario | Contraseña | Empresa |
|---------|-----------|---------|
| insersa | Insersa123 | Insersa |
| royman  | Royman123  | Royman  |
| mimese  | Mimese123  | Mimese  |
| ventura | Ventura123 | Ventura |

Cada proveedor ve únicamente las líneas de sus solicitudes.

## Tecnología

- HTML5 + CSS3 + JavaScript vanilla
- Sin framework, sin dependencias externas, sin build step
- Persistencia en `localStorage` del navegador (los datos se comparten entre las dos apps si se abren en el mismo navegador)

# Portal de Gestión de Mantenimiento — Atalaya Mining

Prototipo estático del sistema de gestión de solicitudes de personal y maquinaria auxiliar de mantenimiento. Funciona directamente en el navegador sin servidor ni instalación.

## Cómo ejecutar

Abre cualquiera de los archivos `.html` directamente en el navegador (doble clic o arrastrar a Chrome/Edge). No requiere servidor web.

## Aplicaciones

### App interna (staff Atalaya)

| Archivo | Función |
|---------|---------|
| `solicitud_personal.html` | Crear y editar solicitudes de personal |
| `tabla_solicitudes.html` | Gestionar solicitudes de personal: ver, aprobar/rechazar, registrar Real |
| `solicitud_generadores.html` | Crear registros de solicitud de maquinaria auxiliar |
| `tabla_generadores.html` | Gestionar registros de maquinaria: ver, editar todos los campos, seguimiento de ciclo |

### Portal externo (proveedores)

| Archivo | Función |
|---------|---------|
| `login.html` | Acceso al portal (credenciales por empresa) |
| `portal.html` | Pantalla de bienvenida tras el login |
| `solicitudes.html` | Ver solicitudes asignadas y confirmar personal disponible |

## Flujo de trabajo — Personal

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

### Estados de cada línea (personal)

| Estado | Descripción |
|--------|-------------|
| No enviado | Creada pero no enviada al proveedor |
| Enviado | Enviada al proveedor, esperando confirmación |
| Conf. ext. | Proveedor ha confirmado — pendiente revisión interna |
| Aprobado | Staff aprueba la confirmación — se puede rellenar Real |
| Rechazado | Staff rechaza la confirmación (definitivo) |
| Cancelada | Línea cancelada por el staff |

## Flujo de trabajo — Maquinaria auxiliar

```
1. Staff crea registro en solicitud_generadores.html
   (empresa, zona, día solicitado, potencia, medios propios, etc.)
2. Registro queda en estado "Sin entregar"
3. Staff completa datos de entrega desde tabla_generadores.html
   (Nº equipo fab., Nº equipo Atalaya, Gasoil S, Día entrega, Baja, Recogida, kW entregada)
4. Registro avanza a "En ciclo" → "Completado"
```

### Estados de cada línea (maquinaria)

| Estado | Descripción |
|--------|-------------|
| Pendiente | Sin día de entrega registrado |
| En ciclo | Tiene día de entrega pero no de recogida |
| Completada | Tiene día de recogida — ciclo cerrado |

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
- Persistencia en `localStorage` del navegador (los datos se comparten entre las apps si se abren en el mismo navegador)

| Clave localStorage | Contenido |
|--------------------|-----------|
| `sdi_solicitudes` | Solicitudes de personal |
| `sdi_generadores` | Registros de maquinaria auxiliar |

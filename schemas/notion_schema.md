# Domus Propiedades · Esquema Notion ("cerebro relacional")

Espacio de trabajo: **Domus Propiedades**
Página contenedora: **Domus · Cerebro IA** (`4124912dc9b6806197a8c21f4099bb28`)

> Los nombres de propiedades son exactos (sin tildes donde no las llevan). El flujo de n8n escribe por nombre: no renombrar.
> En la API de Notion, `rich_text` aparece como `text` en algunas respuestas del MCP; es el mismo tipo.

## DATABASE IDs

| Base | DATABASE ID | Data source ID (referencia) |
|---|---|---|
| Contactos | `c5fdd9d354514d53acf8e5a5af000738` | `d2c0c59a-c7f1-402a-a4c1-37a47c48dcb0` |
| Consultas | `02a9debef55e4201a3635dd5000d3be1` | `7dfce926-90e9-44f1-bf30-3ca7996bcf13` |
| Base de Conocimiento | `d163c668fc334eea9acf0ca3a1ab8e6f` | `acdbac53-5bb6-48e8-bd3c-5a62d54d9c54` |
| Log de Ejecuciones | `b7127b507ece4132a551b5021affa6bb` | `ad0f5333-803c-42c3-b8d6-62c1fb1475d2` |

## Diagrama de relaciones

```
Contactos 1─N Consultas N─N Base de Conocimiento
                  │
                  1
                  │
                  N
          Log de Ejecuciones

Contactos.Consultas            <──> Consultas.Contacto
Consultas.Articulos usados     <──> Base de Conocimiento.Usado en consultas
Consultas.Errores              <──> Log de Ejecuciones.Consulta
```

Nota: Notion no impone cardinalidad en las relaciones; el "1" de Contacto y de Consulta es una convención que respeta el flujo de n8n (envía un único ID).

## Contactos

| Propiedad | Tipo | Opciones / destino |
|---|---|---|
| Nombre | title | |
| Email | email | |
| Tipo de contacto | select | Interesado, Inquilino, Propietario, Otro |
| Alta | created_time | |
| Consultas | relation | → Consultas (inversa: "Contacto") |

## Consultas

| Propiedad | Tipo | Opciones / destino |
|---|---|---|
| Asunto | title | |
| Estado | select | Pendiente, Procesado por IA, Aprobado por Humano, Enviado, Rechazado, Derivado a humano, Datos incompletos, Error |
| Categoria | select | Comercial, Mantenimiento, Administrativa, Otro |
| Urgencia | select | Alta, Media, Baja |
| Mensaje minimizado | rich_text | |
| Datos redactados | multi_select | DNI, CBU/CVU, Telefono, Email, Tarjeta |
| Resumen IA | rich_text | |
| Borrador IA | rich_text | |
| Contacto | relation | → Contactos (inversa: "Consultas") |
| Articulos usados | relation | → Base de Conocimiento (inversa: "Usado en consultas") |
| Recibida | date | |
| Respondida | date | |
| Tiempo de respuesta (min) | formula | `dateBetween(prop("Respondida"), prop("Recibida"), "minutes")` |
| Tokens IA | number | |
| Modelo IA | rich_text | |
| ID ejecucion | rich_text | |
| Gmail Thread ID | rich_text | |
| Errores | relation | → Log de Ejecuciones (inversa: "Consulta") |

## Base de Conocimiento

| Propiedad | Tipo | Opciones / destino |
|---|---|---|
| Tema | title | |
| Categoria | select | Comercial, Mantenimiento, Administrativa, Otro |
| Contenido | rich_text | |
| Verificado | checkbox | |
| Ultima revision | date | |
| Usado en consultas | relation | → Consultas (inversa: "Articulos usados") |

Datos semilla: 14 artículos (13 verificados, 1 sin verificar: "Promoción de verano (borrador, no publicar)"), todos con Ultima revision = 2026-11-01.

## Log de Ejecuciones

| Propiedad | Tipo | Opciones / destino |
|---|---|---|
| Evento | title | |
| Tipo | select | Error IA, Error API, Error envio, Error no controlado, Timeout HITL |
| Workflow | select | WF01 Atencion, WF99 Errores |
| Nodo | rich_text | |
| Detalle | rich_text | |
| ID ejecucion | rich_text | |
| Fecha | date | |
| Consulta | relation | → Consultas (inversa: "Errores") |

# Prompt para generar el cerebro relacional en Notion

**Herramienta:** Claude Code (plan Pro) con el servidor MCP oficial de Notion.
**Uso:** abrir una terminal en la carpeta del repo, ejecutar `claude` y pegar el prompt completo.
**Por qué así:** la rúbrica pide un "esquema de tablas vinculadas generado mediante instrucciones a la IA". Omni AI es la función de Airtable; en Notion el equivalente es dar instrucciones a una IA con acceso al workspace. Acá se usa Claude Code con el conector de Notion.

---

## Prompt

```
Actuá como arquitecto de datos. Usando las herramientas de Notion (MCP), construí el "cerebro relacional" de un sistema de automatización para Domus Propiedades, una inmobiliaria ficticia. Todo debe quedar DENTRO de la página existente "Domus · Cerebro IA" (buscala primero; no crees otra página raíz).

REGLA CRÍTICA: usá EXACTAMENTE los nombres de propiedades indicados, sin tildes donde no las tienen, respetando mayúsculas y espacios. Un flujo de n8n escribe en estas propiedades por nombre a través de la API: cualquier diferencia rompe la integración.

== BASE 1: "Contactos" ==
- Nombre → title
- Email → email
- Tipo de contacto → select: Interesado, Inquilino, Propietario, Otro
- Alta → created_time
- Consultas → relation con "Consultas" (bidireccional; del otro lado se llama "Contacto")

== BASE 2: "Consultas" ==
- Asunto → title
- Estado → select: Pendiente, Procesado por IA, Aprobado por Humano, Enviado, Rechazado, Derivado a humano, Datos incompletos, Error
- Categoria → select: Comercial, Mantenimiento, Administrativa, Otro
- Urgencia → select: Alta, Media, Baja
- Mensaje minimizado → rich_text
- Datos redactados → multi_select: DNI, CBU/CVU, Telefono, Email, Tarjeta
- Resumen IA → rich_text
- Borrador IA → rich_text
- Contacto → relation con "Contactos" (la misma relación bidireccional de arriba)
- Articulos usados → relation con "Base de Conocimiento" (bidireccional; del otro lado se llama "Usado en consultas")
- Recibida → date
- Respondida → date
- Tiempo de respuesta (min) → formula: dateBetween(prop("Respondida"), prop("Recibida"), "minutes")
- Tokens IA → number
- Modelo IA → rich_text
- ID ejecucion → rich_text
- Gmail Thread ID → rich_text
- Errores → relation con "Log de Ejecuciones" (bidireccional; del otro lado se llama "Consulta")

== BASE 3: "Base de Conocimiento" ==
- Tema → title
- Categoria → select: Comercial, Mantenimiento, Administrativa, Otro
- Contenido → rich_text
- Verificado → checkbox
- Ultima revision → date
- Usado en consultas → relation (el lado inverso de "Articulos usados")

== BASE 4: "Log de Ejecuciones" ==
- Evento → title
- Tipo → select: Error IA, Error API, Error envio, Error no controlado, Timeout HITL
- Workflow → select: WF01 Atencion, WF99 Errores
- Nodo → rich_text
- Detalle → rich_text
- ID ejecucion → rich_text
- Fecha → date
- Consulta → relation (el lado inverso de "Errores")

== DATOS SEMILLA: cargá estas 14 filas en "Base de Conocimiento" ==
(Tema | Categoria | Verificado | Contenido). En todas, "Ultima revision" = 2026-11-01.

1. Requisitos para alquilar | Comercial | sí | Para alquilar con Domus se solicita: DNI del titular, constancia de ingresos de los últimos 3 meses (recibos de sueldo, monotributo o certificación contable) y una garantía: seguro de caución o garantía propietaria. Los ingresos demostrables deben ser al menos 3 veces el valor del alquiler. La documentación se envía por email a administración una vez reservada la propiedad.
2. Visitas a propiedades | Comercial | sí | Las visitas se coordinan de lunes a sábados de 10 a 18 h, con al menos 24 h de anticipación. Un asesor confirma día y horario por email. En propiedades ocupadas, la visita depende de la disponibilidad del inquilino actual.
3. Reserva de propiedades | Comercial | sí | La reserva de un alquiler se realiza con un pago equivalente al 10% del primer mes; en compras, con USD 1.000. Se descuenta del total al firmar. Si el propietario rechaza la operación, la reserva se devuelve completa en 5 días hábiles.
4. Catálogo vigente de alquileres (noviembre 2026) | Comercial | sí | DOM-101: 2 ambientes en Palermo, 45 m², ARS 850.000/mes + expensas ARS 120.000, apto mascotas. DOM-102: 3 ambientes en Caballito, 70 m², ARS 1.100.000/mes + expensas ARS 150.000, con balcón. DOM-103: monoambiente en Almagro, 30 m², ARS 600.000/mes + expensas ARS 80.000. DOM-104: 4 ambientes en Belgrano, 110 m², ARS 1.900.000/mes + expensas ARS 250.000, cochera incluida.
5. Catálogo vigente de ventas (noviembre 2026) | Comercial | sí | DOM-201: 2 ambientes en Villa Crespo, 50 m², USD 105.000. DOM-202: 3 ambientes en Núñez, 80 m², USD 185.000, apto crédito. DOM-203: PH de 4 ambientes en Villa Urquiza, 120 m², USD 230.000, con terraza.
6. Honorarios de la inmobiliaria | Comercial | sí | Alquileres: los honorarios de Domus están a cargo del propietario; el inquilino no abona comisión. Ventas: 3% del valor de la operación a cargo del comprador. Ante dudas sobre una operación puntual, un asesor confirma los valores por escrito.
7. Reclamos urgentes de mantenimiento | Mantenimiento | sí | Se consideran urgentes: pérdidas de gas, pérdidas de agua importantes o inundaciones, cortes eléctricos totales y problemas de seguridad en accesos. Domus deriva el caso a un técnico de guardia en un plazo máximo de 4 horas. Ante olor a gas, el inquilino debe cerrar la llave de paso, ventilar el ambiente y llamar a la línea de emergencias de la distribuidora.
8. Reclamos no urgentes de mantenimiento | Mantenimiento | sí | Los desperfectos que no afectan la habitabilidad (canillas que gotean, persianas trabadas, humedad leve) se coordinan dentro de las 72 h hábiles. Un técnico contacta al inquilino para acordar la visita. Se recomienda enviar fotos del problema para agilizar el diagnóstico.
9. Responsabilidades de reparación | Mantenimiento | sí | Las reparaciones menores y el mantenimiento de uso (lámparas, burletes, destapaciones por mal uso) están a cargo del inquilino. Las reparaciones estructurales, de instalaciones de gas, agua y electricidad, y de artefactos provistos con la propiedad están a cargo del propietario y las gestiona Domus.
10. Pago del alquiler | Administrativa | sí | El alquiler se paga del 1 al 10 de cada mes por transferencia bancaria a la cuenta indicada en el contrato. El comprobante se envía a administración por email. Los pagos fuera de término generan el interés diario pactado en el contrato.
11. Actualización del valor del alquiler | Administrativa | sí | El valor se actualiza según el índice y la periodicidad pactados en cada contrato. Domus informa el nuevo monto por email con al menos 15 días de anticipación.
12. Rescisión anticipada del contrato | Administrativa | sí | El inquilino puede rescindir el contrato avisando por escrito con 30 días de anticipación. Las condiciones económicas dependen de lo pactado en cada contrato; un asesor las confirma por escrito antes de cualquier decisión.
13. Horarios y canales de atención | Administrativa | sí | Atención de lunes a viernes de 9 a 18 h por email. Las consultas recibidas fuera de horario se responden el siguiente día hábil, salvo urgencias de mantenimiento.
14. Promoción de verano (borrador, no publicar) | Comercial | NO | Borrador interno: 50% de descuento en el primer mes para contratos firmados en diciembre. Pendiente de aprobación de los propietarios.

No cargues filas en Contactos, Consultas ni Log de Ejecuciones: las genera el flujo de n8n.

== VERIFICACIÓN Y SALIDA ==
1. Después de crear todo, volvé a leer cada base y verificá que los nombres y tipos de propiedades coinciden EXACTAMENTE con lo pedido. Corregí cualquier diferencia.
2. Para cada base, obtené el DATABASE ID (los 32 caracteres hexadecimales de la URL de la base, no el ID de la vista ni el del data source).
3. Creá el archivo schemas/notion_schema.md con: una tabla por base (propiedad, tipo, opciones o destino de la relación), un diagrama de relaciones en texto (Contactos 1─N Consultas N─N Base de Conocimiento; Consultas 1─N Log de Ejecuciones) y la lista de DATABASE IDs.
4. Creá el archivo schemas/notion_schema.json con la misma información en JSON: {"databases":[{"name","database_id","properties":[{"name","type","options"|"relation_to"|"formula"}]}]}.
5. Al final mostrame en pantalla los 4 DATABASE IDs, uno por línea, con el nombre de la base.
```

---

## Evidencias

- Conversación completa en la terminal, con el prompt y la respuesta → `evidencias/fase2-notion/01_conversacion_claude_code.txt`
- Las 4 bases creadas en Notion → `evidencias/fase2-notion/02_bases_creadas.png`
- Una base con sus relaciones visibles → `evidencias/fase2-notion/03_relaciones.png`
- Base de Conocimiento con los 14 artículos semilla → `evidencias/fase2-notion/04_base_conocimiento.png`

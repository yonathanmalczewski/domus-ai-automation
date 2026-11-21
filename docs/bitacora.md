# Bitácora del proyecto

Registro de decisiones, configuraciones e incidentes en orden cronológico. Los incidentes se documentan tal cual ocurrieron, con su causa y su solución.

Formato de cada entrada:

- **Fecha · Fase**
- **Qué se hizo**
- **Decisión y por qué** (si aplica)
- **Incidente → causa → solución** (si aplica)
- **Evidencia** (ruta de la captura en `evidencias/`)

---

## 2026-11-02 · Fase 0 · Definiciones iniciales

**Decisiones**

- **Caso de uso:** Domus Propiedades (inmobiliaria y administradora de alquileres ficticia). Triage de consultas por email y respuesta con información verificada (RAG). Retoma el caso trabajado en el curso de Introducción a la IA.
- **Orquestador:** n8n self-hosted con Docker. Motivo: gratis, sin vencimiento y sin límite de ejecuciones. Contra: la credencial de Gmail requiere crear un cliente OAuth propio en Google Cloud.
- **Canales:** Gmail (comunicación con el cliente) + Slack (aprobación humana y alertas internas). WhatsApp queda fuera del alcance.
- **Base de datos:** Notion.
- **IA:** Anthropic Claude. Haiku para tareas mecánicas, Sonnet para redacción con contexto denso, Message Batches API para el proceso masivo nocturno.

**Ajustes respecto de la consigna**

- La rúbrica pide el esquema "generado mediante instrucciones a la IA (Omni AI)". Omni AI es una función de Airtable. Como el proyecto usa Notion, el equivalente es **Notion AI**. Se documenta el prompt usado y el resultado.
- La rúbrica cita GPT-4o-mini como ejemplo de modelo económico. Se eligió **Claude Haiku** para mantener un solo proveedor; GPT-4o-mini se incluye en el cuadro comparativo como alternativa evaluada.

**Riesgos conocidos**

- Con la app de Google en estado *Testing* y usuario *External*, el token de Gmail vence cada 7 días. Hay que reconectar la credencial antes de las pruebas finales y de grabar el video.

---

## 2026-11-08 · Replanificación por tiempo y presupuesto

**Restricciones:** entrega en el día (menos de 3 h de trabajo) y sin presupuesto para cargar crédito en la API de Anthropic.

**Decisiones**

- **Motor de IA en ejecución: Google Gemini (tier gratuito)**, llamado por HTTP con salida JSON forzada (`responseSchema`) y conteo de tokens (`usageMetadata`). La suscripción Claude Pro no incluye acceso a la API, y usar su token de sesión en herramientas de terceros como n8n está prohibido por los términos de Anthropic. Claude se usa en la etapa de diseño (Claude Code con el conector MCP de Notion genera el esquema relacional) y queda como el modelo objetivo de producción en la matriz de costos. La arquitectura no depende del modelo: cambiar de proveedor implica modificar la URL y el cuerpo de dos nodos HTTP.
- **Trigger: n8n Form Trigger** (formulario web de consultas) en lugar de Gmail Trigger. Evita crear un cliente OAuth en Google Cloud, permite probar al instante y da datos estructurados para validar.
- **Salida al cliente: email por SMTP de Gmail** con contraseña de aplicación. **Hilos:** cada consulta tiene su propio hilo en Slack (`thread_ts`) con resumen, aprobación y resultado.
- **Alcance reducido:** 4 bases en Notion (Contactos, Consultas, Base de Conocimiento, Log de Ejecuciones). El proceso por lotes (Batches) se diseña y cuantifica en el documento, pero no se implementa.
- **Nombres de propiedades sin tildes** en Notion (por ejemplo `Categoria`) para evitar fallas de coincidencia exacta en la API.

---

## 2026-11-08 · Cambio de orquestador: n8n local → n8n Cloud

**Decisión:** n8n Cloud (prueba de 14 días con funciones Pro y hasta 1.000 ejecuciones) en lugar de n8n self-hosted con Docker.

**Por qué**

- **Riesgo de setup:** en Windows, Docker Desktop requiere WSL2, virtualización habilitada y reinicio. Con menos de 3 h disponibles, una falla en ese paso comprometía la entrega. En Cloud, la instancia queda lista en minutos y sin instalar nada.
- **URLs públicas:** con n8n local, el formulario y los botones de aprobación de Slack apuntan a `localhost` y solo funcionan desde la misma PC. En Cloud, el formulario se puede abrir y la aprobación humana se puede dar desde cualquier dispositivo, incluido el celular.
- **Vencimiento de la prueba:** no afecta la entrega. Los entregables evaluados (JSON en el repo, dashboard en Notion, capturas y video) no dependen de que la instancia siga activa, y n8n permite descargar los flujos hasta 90 días después.

**Impacto:** ninguno en los flujos ni en los nombres de las credenciales. El riesgo de vencimiento del token de Google OAuth (registrado el 02/11) ya no aplica, porque el envío se hace por SMTP.

---

## 2026-11-08 · Salida al cliente: SMTP → nodo Gmail con OAuth

**Decisión:** el email al cliente se envía con el nodo **Gmail** de n8n Cloud, autenticado con *Sign in with Google* (credencial `Domus · Gmail`), en lugar de SMTP con contraseña de aplicación.

**Por qué:** la contraseña de aplicación exige la verificación en 2 pasos de Google y suma un secreto más para custodiar. En n8n Cloud, el OAuth de Google está gestionado por la plataforma: no hace falta un proyecto en Google Cloud y el acceso se revoca desde la cuenta de Google. Se descartó la alternativa de responder solo por Slack, porque perdía el contacto real con el cliente y la salida multicanal.

**Mínimo privilegio en la autorización de Google:** de los 6 permisos que pide la app de n8n Cloud, solo se otorgó *"Administrar borradores y enviar correos electrónicos"* (`gmail.compose`). Ese permiso cubre las dos llamadas que hace el nodo: `users/me/profile` para el remitente y `messages/send`. Se rechazaron el acceso total con borrado permanente (`mail.google.com`), la lectura de toda la casilla (`gmail.modify`), las etiquetas y los permisos de complementos. Tampoco se otorgó acceso continuo: Google lo hace vencer automáticamente a los 6 meses (07/05/2027). Evidencia: `evidencias/fase0-setup/03_gmail_permisos_minimos.png`.

**Mejora asociada:** el nodo Gmail devuelve el `threadId` del mensaje. Se guarda en Notion (`Gmail Thread ID`) y se informa en el hilo de Slack de la consulta. Así cada consulta queda trazada en los dos canales (mapeo de hilos que pide la consigna).

---

## 2026-11-08 al 12 · Fase 0 · Configuración de cuentas

- [x] Instancia de n8n Cloud activa → `evidencias/fase0-setup/01_n8n_cloud.png`
- [x] Credencial `Domus · Notion` creada. La conexión de Notion tiene solo capacidades de contenido (leer, actualizar, insertar), sin acceso a información de usuarios, y acceso únicamente a las 2 páginas de Domus.
- [x] Credencial `Domus · Slack` creada (app *Domus Bot* desde manifiesto, 4 scopes) y bot invitado a los 3 canales
- [x] Credencial `Domus · Gemini` (Header Auth `x-goog-api-key`) creada con una clave exclusiva para este proyecto, revocable por separado
- [x] Credencial `Domus · Gmail` (Gmail OAuth2, *Sign in with Google*) conectada → `evidencias/fase0-setup/02_credenciales.png`
- [x] Conector MCP de Notion autorizado en Claude Code
- [x] Repo creado en GitHub (`domus-ai-automation`, público), subido el 20/11 después de revisar que ninguna captura muestre el email personal

**Incidentes**

- *Slack:* la pantalla de creación de apps cambió. Se usó la opción **Paste a manifest**, con nombre, scopes y rotación de tokens definidos en JSON. Los pasos de *Slack CLI* que ofrece al final no aplican porque no se programa una app.
- *Notion:* la creación de integraciones pasó a **Herramientas para desarrolladores → Conexiones → Nueva conexión** (método *Token de API*).
- *Credenciales:* la de Notion se guardó como `Domus · Notion de n8n` y se renombró a `Domus · Notion`, para que los flujos importados la asignen solos por nombre.

---

## 2026-11-12 · Fase 2 · Cerebro en Notion con Claude Code

**Qué se hizo:** Claude Code, con el conector MCP oficial de Notion, creó las 4 bases relacionadas y cargó los 14 artículos semilla a partir de `prompts/00_prompt_notion_claude_code.md`. Después verificó nombres y tipos y generó `schemas/notion_schema.md` y `schemas/notion_schema.json`. Evidencia: `evidencias/fase2-notion/01_conversacion_claude_code.txt`.

**Incidentes**

- *Claude Code no instalado:* el comando `claude` no se reconocía en PowerShell. Se instaló con el instalador oficial (`irm https://claude.ai/install.ps1 | iex`) y se agregó `%USERPROFILE%\.local\bin` al PATH del usuario.
- *Workspace equivocado:* la cuenta de Notion tiene 3 workspaces y la primera autorización del MCP quedó en *Notion de Yonathan Malczewski*. Se detectó con un prompt de solo lectura que buscaba la página raíz **antes** de crear nada. Se limpió la autenticación (`/mcp → Clear authentication`) y se volvió a autorizar con *Domus Propiedades* activo.
- *Ubicación de Contactos:* se pidió crear *Contactos* dentro de la página privada, pero quedó en la raíz. Se movió a mano a `Domus · Datos personales (privado)`, para que la base con datos personales no quede publicada junto al dashboard.

---

## 2026-11-17 · Fase 3 · Importación de los flujos

- WF99 y WF01 se importaron desde `workflows/`, con las credenciales asignadas automáticamente por nombre y los IDs de Notion y Slack ya cargados en el nodo *Config*.
- *Incidente:* el WF99 no se podía elegir como *Error workflow* del WF01: aparecía deshabilitado con una advertencia. **Causa:** en n8n 2.x, un flujo tiene que estar publicado para usarse como manejador de errores. **Solución:** se publicó el WF99 y se asignó en *Settings* del WF01.
- No se habilitó el acceso MCP a los flujos, que ofrece la *Production Checklist* de n8n: no es necesario y reduce la superficie de exposición.

---

## 2026-11-18 · Fase 4 · Incidente en la primera ejecución: timeout de Gemini

- **Qué pasó:** la ejecución #1 (prueba 1, consulta comercial) terminó como *Succeeded* en 1 min 23 s, pero no pidió aprobación en Slack ni envió el email.
- **Causa:** el nodo `IA · Redactar respuesta (Gemini)` superó el límite de 60 s (`timeout of 60000ms exceeded`) con el modelo `gemini-flash-latest`. En el tier gratuito, ese modelo, que razona antes de responder, tardó más que el límite configurado. El clasificador (`gemini-flash-lite-latest`) respondió sin problemas.
- **Comportamiento del sistema:** la ruta de error controlada funcionó tal como se diseñó. La falla quedó registrada en *Log de Ejecuciones*, la consulta pasó a estado *Error*, llegó la alerta a #alertas-sistema y **no se contactó al cliente**. Este incidente real queda como evidencia de la prueba 6 (falla de la API de IA), así que no hace falta forzar esa falla.
- **Corrección:** `modelo_redactor` pasa a `gemini-flash-lite-latest` y el timeout de los dos nodos de IA sube de 60 s a 120 s. En producción, la redacción quedaría a cargo de Claude Sonnet (ver estrategia de costos).
- **Segundo hallazgo:** el *Asunto* de la consulta mostraba la hora en UTC (23:18 en lugar de 20:18), porque el flujo importado no tomó la zona horaria. Se configuró `America/Argentina/Buenos_Aires` en *Settings* del WF01.
- **Verificación:** la ejecución #2 (18/11, 21:10) completó el camino feliz. El borrador salió con semáforo 🟢, confianza *Alta*, 3 artículos de la base citados y sin alertas del guardrail. Se aprobó en Slack, el email llegó con la aclaración de IA al pie y Notion quedó en *Enviado*, con el `Gmail Thread ID` registrado y el teléfono redactado. El contacto se reconoció como recurrente gracias a la consulta fallida anterior: la memoria también funciona.

---

## 2026-11-18 · Fase 8 · Prueba de estrés: 9 ejecuciones

Se corrieron 9 ejecuciones con el flujo publicado: 4 caminos felices y 5 infelices. Todas terminaron de forma controlada y ningún email salió sin aprobación. El detalle de cada caso (mensaje, resultado esperado y obtenido, tokens) está en `data/casos_de_prueba.csv`.

| Ej. | Prueba | Resultado |
|---|---|---|
| #1 | 6 · Falla de la IA | Incidente real del timeout (ver arriba): *Error*, registrado y alertado |
| #2 | Camino feliz (verificación) | *Enviado* |
| #3 | 1 · Camino feliz | *Enviado*, teléfono redactado, `Gmail Thread ID` guardado |
| #4 | 2 · Memoria del contacto | Contacto reconocido como recurrente; *Enviado* |
| #5 | 3 · Urgencia (olor a gas) | Alerta inmediata en #mantenimiento-urgente; *Enviado* |
| #6 | 4 · Anti-alucinación y rechazo | El borrador no usó la promoción no verificada ni inventó precios; *Rechazado* |
| #7 | 5 · Datos incompletos | No se llamó a la IA; *Datos incompletos* y aviso en Slack |
| #8 | 7 · Anti-bucle | El filtro descartó el ítem; no se creó ningún registro |
| #9 | 8 · Inyección de instrucciones | Clasificada *Otro* y derivada a una persona, sin borrador |

---

## 2026-11-19 · Fase 7 · Dashboard público en Notion

- Página *Dashboard Domus* con 4 bloques de vistas enlazadas: gráfico *Salud del flujo*, tabla de KPIs con cálculos al pie, tablero *Pipeline* por estado y *Errores registrados* (Log ordenado por fecha).
- *Incidente:* Notion no ofrece "porcentaje por grupo" para propiedades de selección. **Solución:** columnas de fórmula `% Error` y `% Enviado` (`if(prop("Estado") == "Error", 100, 0)`) con *Promedio* al pie.
- Publicado con *Publicar en la web* y verificado en una ventana privada: https://bead-canid-cb8.notion.site/Dashboard-Domus-41d4912dc9b680969b00d02ff786017f
- Privacidad: la columna *Contacto* está oculta en todas las vistas y la base *Contactos* no se publica.
- Valores al cierre de la prueba de estrés: 8 consultas, 3,25 min de tiempo medio de respuesta, 8.734 tokens, 12,5 % de error y 50 % de envío.

---

## 2026-11-19 · Documento del proyecto y saneamiento de los flujos

- `docs/Proyecto_Final_Domus.pdf` (14 páginas) cubre los 5 criterios de la rúbrica, la prueba de estrés y los anexos. El diagrama de arquitectura está en `docs/diagramas/` (SVG y PNG). Los precios de la estrategia de costos se consultaron el 19/11/2026 en las páginas oficiales de Anthropic, Google y OpenAI.
- `schemas/esquemas_transferencia.json` formaliza como JSON Schema los 8 contratos de datos entre componentes.
- WF01 y WF99 se exportaron desde n8n (*Export JSON*). En el WF01, el email interno de Domus del nodo *Config* se reemplazó por `PEGAR_TU_GMAIL@gmail.com`. Búsqueda de `xoxb`, `ntn_`, `AIza`, `secret_` y `Bearer`: 0 resultados. Las credenciales figuran solo por nombre.
- Se difuminó el email personal en `evidencias/fase5-errores/t7_ejecucion.png`.

---

## 2026-11-20 · Datos de prueba y video demo

- `data/` se completó con `base_conocimiento_semilla.csv` (los 14 artículos cargados por Claude Code) y `casos_de_prueba.csv` (las 8 pruebas del 18/11).
- **Video demo:** se grabaron 6 tomas con 5 pruebas en vivo y contactos nuevos: #10 camino feliz (*Enviado*), #11 urgencia por inundación (alerta inmediata en #mantenimiento-urgente), #13 rechazo humano (*Rechazado*), #14 datos incompletos y #15 inyección de instrucciones (derivada a una persona).
- *Incidente (error humano):* en la primera toma del rechazo se hizo clic en *Aprobar y enviar* en lugar de *Rechazar* (ejecución #12). El email llegó a la casilla propia, por el alias de prueba, así que no salió nada hacia afuera. La consulta queda en Notion como *Enviado*, porque es un registro real. La toma se repitió con un contacto nuevo (#13) para que la memoria no lo tratara como recurrente.
- **Edición:** sin audio, con carteles por escena. El email personal (5 formularios y el encabezado de Gmail) y dos enlaces de aprobación de n8n, que aparecían al pasar el mouse por los botones de Slack, se taparon. Antes de subirlo se revisó el video final completo: no quedan visibles emails, enlaces de aprobación ni claves.
- Dashboard al cierre de la grabación: 14 consultas, 2,33 min de tiempo medio de respuesta, 15.896 tokens, 7,14 % de error y 42,86 % de envío.
- Video (2:33): https://drive.google.com/file/d/1hTxj5kD1e5U13y5Ah7OOcYCBGrPa76un/view?usp=sharing


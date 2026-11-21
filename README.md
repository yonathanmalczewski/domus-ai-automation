# Domus Propiedades · Ecosistema de Automatización IA Autónomo

Proyecto final del curso **AI Automation** (Diplomatura en Data Science · Coderhouse).

> **Enlaces rápidos:** [Dashboard público en Notion](https://bead-canid-cb8.notion.site/Dashboard-Domus-41d4912dc9b680969b00d02ff786017f) · [Video demo (2:33, sin audio)](https://drive.google.com/file/d/1hTxj5kD1e5U13y5Ah7OOcYCBGrPa76un/view?usp=sharing) · [Documento del proyecto (PDF)](docs/Proyecto_Final_Domus.pdf)

## 1. Caso de uso

**Domus Propiedades** (empresa ficticia) es una inmobiliaria y administradora de alquileres. Recibe consultas mezcladas: interesados en alquilar o comprar, reclamos de mantenimiento de inquilinos y consultas administrativas.

El ecosistema hace este recorrido con cada consulta que llega por el formulario web:
1. Minimiza los datos personales.
2. Recupera la memoria del contacto en Notion.
3. Clasifica la consulta con IA.
4. Redacta una respuesta basada **solo** en información verificada de la empresa (RAG sobre Notion).
5. Pide aprobación humana por Slack, en un hilo por consulta.
6. Si se aprueba, responde al cliente por email.

Todo queda registrado en Notion y se monitorea desde un dashboard público.

## 2. Stack

| Categoría | Herramienta | Rol en el sistema |
|---|---|---|
| Orquestador | n8n Cloud (prueba con funciones Pro) | WF01 (atención en tiempo real) y WF99 (manejador global de errores) |
| Base de datos | Notion | Cerebro relacional, memoria del sistema, base de conocimiento (RAG), log de ejecuciones y dashboard |
| Procesamiento IA | Google Gemini (en ejecución) · Claude (en diseño) | Gemini 3.5 Flash-Lite clasifica y redacta, con salida JSON forzada (el redactor pasó de Flash a Flash-Lite tras el incidente del 18/11). Claude Code, con el MCP de Notion, generó el esquema relacional; Claude es el modelo objetivo de producción en la matriz de costos |
| Canales | Formulario n8n · Slack · Gmail (OAuth) | Formulario: entrada. Slack: aprobación HITL, alertas y un hilo por consulta. Gmail: respuesta al cliente |

> El cambio de Claude a Gemini como motor en ejecución está justificado en `docs/bitacora.md` (restricción de presupuesto: la suscripción Claude Pro no incluye acceso a la API).

## 3. Entregables (mapeo con la rúbrica)

| Criterio de evaluación | Dónde está |
|---|---|
| Mapa de arquitectura del sistema | `docs/Proyecto_Final_Domus.pdf` · sección 1 (fuente en `docs/diagramas/`) |
| Manual operativo de estructuras de datos | PDF · sección 2 + `schemas/` |
| Estrategia de optimización de costos | PDF · sección 3 |
| Malla de seguridad, privacidad y resiliencia | PDF · sección 4 |
| Dashboard de control ejecutivo | [Dashboard público en Notion](https://bead-canid-cb8.notion.site/Dashboard-Domus-41d4912dc9b680969b00d02ff786017f) · PDF · sección 5 |

**Archivos técnicos de respaldo**

- Lógica de los flujos: `workflows/*.json` (exportados de n8n, sin credenciales)
- Base de datos en modo lectura: [dashboard público de Notion](https://bead-canid-cb8.notion.site/Dashboard-Domus-41d4912dc9b680969b00d02ff786017f) (vistas enlazadas a *Consultas* y *Log de Ejecuciones*; la base *Contactos* no se publica)
- Evidencias: `evidencias/` (capturas por fase)
- Video demo: [Google Drive](https://drive.google.com/file/d/1hTxj5kD1e5U13y5Ah7OOcYCBGrPa76un/view?usp=sharing) (2:33, sin audio, con carteles; email y enlaces de aprobación tapados)
- Casos de prueba y base de conocimiento semilla: `data/`

## 4. Estructura del repositorio

```
domus-ai-automation/
├── README.md
├── docs/
│   ├── Proyecto_Final_Domus.pdf   # documento principal (5 criterios)
│   ├── bitacora.md                # decisiones e incidentes, en orden
│   ├── guia_importacion_y_pruebas.md
│   └── diagramas/                 # fuentes del diagrama + PNG
├── workflows/                     # WF01 y WF99 (JSON de n8n)
├── schemas/                       # esquema de Notion + esquemas JSON de transferencia
├── prompts/                       # prompt de Notion (Claude Code), clasificador y redactor
├── data/                          # base de conocimiento semilla (14 artículos) + 8 casos de la prueba de estrés
└── evidencias/                    # capturas organizadas por fase
```

## 5. Seguridad del repositorio

- Ninguna API key, token ni secreto se sube al repo. Las credenciales viven solo en n8n.
- Los JSON exportados de n8n incluyen el **nombre** de la credencial, nunca su valor. Igual se revisan antes de subirlos.
- Todos los datos personales de prueba son ficticios.

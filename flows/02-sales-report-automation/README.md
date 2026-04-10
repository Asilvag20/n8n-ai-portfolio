# Flow 02 — Sales Report Automation

![n8n](https://img.shields.io/badge/n8n-workflow-orange) ![Ollama](https://img.shields.io/badge/LLM-Ollama_llama3.2-blue) ![Google](https://img.shields.io/badge/Data-Google_Drive_%2B_Sheets-green) ![Gmail](https://img.shields.io/badge/Output-Gmail_HTML-red)

Automatización que genera un informe ejecutivo semanal de citas médicas a partir de exports de Doctoralia. Cada lunes a las 8 AM detecta el archivo más reciente en Google Drive, calcula KPIs, genera narrativa con IA local y envía un email HTML profesional.

---

## Problema que resuelve

Los consultorios médicos exportan datos de Doctoralia manualmente y pierden tiempo consolidando métricas cada semana. Este flujo automatiza:

- Detección automática del reporte semanal subido a Drive
- Cálculo de KPIs: ingresos, cancelaciones, origen de citas, top servicios
- Narrativa ejecutiva generada con IA (sin costo de API)
- Entrega automática por email cada lunes antes de empezar la semana

---

## Arquitectura

```
Every Monday 8AM
       │
       ▼
List Drive Folder ──── Google Drive API (carpeta fija)
       │                Busca el archivo más reciente
       ▼
Get Latest File ID ──── Code Node
       │                Extrae fileId del resultado
       ▼
Read Citas Sheet ──── Google Sheets OAuth2
       │               Lee hoja "citas" del archivo dinámico
       ▼
Calculate KPIs ──── Code Node (JavaScript)
       │             Parser COP · filtro encabezados · 8 métricas
       ▼
AI Report Writer ──── Basic LLM Chain
       │    ▲           Prompt estructurado → 4 secciones
       │    └── Ollama llama3.2 (local, sin costo)
       ▼
Build HTML Report ──── Code Node
       │                Email responsive con tablas y KPI cards
       ▼
Send Weekly Report ──── Gmail OAuth2
                         cristian.1003@hotmail.com
```

---

## Nodos

| Nodo | Tipo | Función |
|------|------|---------|
| **Every Monday 8AM** | Schedule Trigger | Dispara lunes 8:00 AM |
| **List Drive Folder** | HTTP Request | Llama Google Drive API v3, filtra por `mimeType=spreadsheet`, ordena por `createdTime desc` |
| **Get Latest File ID** | Code | Extrae `files[0].id` y valida que exista archivo |
| **Read Citas Sheet** | Google Sheets | Lee hoja `citas` usando ID dinámico `{{ $json.fileId }}` |
| **Calculate KPIs** | Code | Filtra filas válidas, parsea precios COP, calcula 8 KPIs |
| **AI Report Writer** | Basic LLM Chain | Genera narrativa ejecutiva en español con prompt estructurado |
| **Ollama llama3.2** | LM Ollama | Modelo local, temperature 0.4 para consistencia |
| **Build HTML Report** | Code | Email HTML responsive: header, KPI cards, tablas, footer |
| **Send Weekly Report** | Gmail | Asunto dinámico con período y totales |

---

## Decisiones de ingeniería

### ¿Por qué Google Drive dinámico en lugar de ID fijo?
Con ID fijo hay que sobrescribir el mismo archivo cada semana (pérdida del histórico). Con la carpeta Drive, se sube un archivo nuevo cada semana y el flow siempre toma el más reciente por `createdTime`. El histórico queda preservado en Drive.

### ¿Por qué HTTP Request para Drive en lugar del nodo nativo?
El nodo nativo de Drive requiere credencial `googleDriveOAuth2Api` separada. Al agregar el scope `drive.readonly` a la credencial existente de Sheets, el HTTP Request reutiliza la misma autenticación OAuth2 sin crear una credencial adicional.

### ¿Por qué Basic LLM Chain y no AI Agent?
El reporte tiene una estructura fija (4 secciones, 350 palabras). Un Agent implicaría overhead de razonamiento y tool-calls innecesarios. El Chain es determinista, rápido y suficiente para este caso.

### Parser de precios colombianos
Doctoralia exporta precios en formato `$ 188.571,00` (punto = miles, coma = decimal). El parser detecta la combinación período+coma y determina cuál es separador de miles usando `lastIndexOf`:

```javascript
if (hasPeriod && hasComma) {
  if (str.lastIndexOf('.') > str.lastIndexOf(',')) {
    // punto después de coma → punto es decimal (formato anglosajón)
    str = str.replace(/,/g, '');
  } else {
    // coma después de punto → punto es miles, coma es decimal (formato COP)
    str = str.replace(/\./g, '').replace(',', '.');
  }
}
```

### Filtro de filas
Doctoralia incluye filas de encabezado o vacías al exportar. El filtro exige que `Fecha` y `Paciente` existan y no sean literalmente los nombres de columna:

```javascript
return fecha && paciente
  && fecha.toLowerCase() !== 'fecha'
  && paciente.toLowerCase() !== 'paciente';
```

---

## Configuración

### Credenciales necesarias

| Credencial | Tipo | Scopes |
|-----------|------|--------|
| Google Sheets account | OAuth2 | `spreadsheets`, `drive.readonly` |
| Gmail account | OAuth2 | `gmail.send` |
| Ollama account | API | `http://host.docker.internal:11434` |

### Carpeta Drive
Crea una carpeta en Google Drive (ej. "Reportes Doctoralia") y actualiza el ID en el nodo **List Drive Folder**:

```
'TU_FOLDER_ID' in parents and trashed=false and mimeType='application/vnd.google-apps.spreadsheet'
```

### Formato del archivo
El archivo de citas debe ser **Google Sheets** (no Excel .xlsx) con una hoja llamada `citas` y estas columnas:

| Columna | Ejemplo |
|---------|---------|
| Fecha | 07/04/2026 |
| Paciente | Juan Pérez |
| Servicios | Consulta General |
| Estado | Confirmada |
| Precio | $ 188.571,00 |
| Aseguradora | Sura |
| Origen de la cita | Doctoralia |

### Flujo de trabajo semanal
```
Domingo noche → Exportar de Doctoralia → Subir a carpeta Drive como Google Sheets
Lunes 8 AM   → Flow detecta archivo → Calcula KPIs → Genera informe → Envía email
```

---

## KPIs calculados

| Métrica | Descripción |
|---------|-------------|
| `totalCitas` | Número de filas válidas |
| `totalIngresos` | Suma de Precio (formato COP) |
| `promedioPorCita` | totalIngresos / totalCitas |
| `tasaCancelacion` | % citas con estado Cancelada/Cancelado |
| `topServicios` | Top 5 servicios por ingresos |
| `topOrigenes` | Distribución por origen de cita |
| `topAseguradoras` | Top 6 aseguradoras/tipo de pago |
| `porEstado` | Distribución completa de estados |

---

## Cómo ejecutar localmente

```bash
# 1. Levantar n8n
docker compose up -d

# 2. Importar el flow
docker cp flows/02-sales-report-automation/flow.json n8n-portfolio:/tmp/flow02.json
docker exec n8n-portfolio n8n import:workflow --input=/tmp/flow02.json

# 3. En n8n UI (http://localhost:5678)
#    - Conectar credencial Google Sheets en nodos "List Drive Folder" y "Read Citas Sheet"
#    - Activar el flow

# 4. Test manual: botón "Test workflow" en n8n
#    (requiere archivo Google Sheets en la carpeta Drive configurada)
```

---

## Impacto de negocio

- **Tiempo ahorrado**: ~45 min/semana de consolidación manual de datos
- **Consistencia**: mismo formato y métricas cada semana, sin errores humanos
- **Costo IA**: $0 — Ollama corre localmente
- **Histórico**: cada reporte semanal preservado en Drive indefinidamente

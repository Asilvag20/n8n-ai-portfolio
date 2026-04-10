# DEMO Guide — Flow 02: Sales Report Automation

Guía para grabar el video demo de este flow. Tiempo objetivo: **60-90 segundos**.

---

## Guión (timestamps)

### [0:00 – 0:10] Contexto del problema
Muestra la carpeta de Google Drive con el archivo de citas subido.
> *"Cada semana se exporta el reporte de Doctoralia y se sube a esta carpeta en Drive."*

### [0:10 – 0:20] El flow en n8n
Abre n8n (`http://localhost:5678`) y muestra el canvas del flow.
> *"Este flow corre cada lunes a las 8 AM, detecta el archivo más reciente en Drive y genera el informe automáticamente."*

Señala los nodos en orden: Schedule → Drive → Sheets → KPIs → AI → HTML → Gmail.

### [0:20 – 0:40] Ejecución en vivo
Haz clic en **"Test workflow"**.

- Muestra **List Drive Folder** ejecutándose — output: nombre del archivo y su ID
- Muestra **Calculate KPIs** — output: totalCitas, totalIngresos, topServicios
- Muestra **AI Report Writer** — output: narrativa ejecutiva generada por Ollama
- Muestra **Send Weekly Report** — output: `messageId` confirmando envío

### [0:40 – 0:60] El email recibido
Cambia a la bandeja de entrada (Hotmail/Outlook).
> *"El informe llega listo para revisión cada lunes."*

Muestra el email abierto:
- Header con gradiente azul/morado y el período
- 4 KPI cards: Total Citas · Ingresos · Promedio · Cancelaciones
- Sección de análisis IA
- Tabla Top Servicios por Ingresos
- Tablas Origen y Estado de citas

### [0:60 – 0:75] Cierre
Vuelve al canvas de n8n.
> *"45 minutos de consolidación manual eliminados. Cero costo de API — Ollama corre localmente."*

---

## Datos de prueba

Asegúrate de tener en la carpeta Drive un Google Sheets con hoja `citas` y al menos estas columnas:

```
Fecha | Paciente | Servicios | Estado | Precio | Aseguradora | Origen de la cita
```

Ejemplo de fila:
```
07/04/2026 | Ana García | Consulta General | Confirmada | $ 188.571,00 | Sura | Doctoralia
```

---

## Tips de grabación

**Herramientas gratuitas (Windows):**
- **Xbox Game Bar** (`Win + G`) — grabación instantánea, sin instalar nada
- **OBS Studio** — control total, ideal para zoom en nodos específicos
- **ShareX** — graba región específica de pantalla

**Configuración recomendada:**
- Resolución: 1920x1080
- Zoom en n8n: 85% para ver todos los nodos en el canvas
- Ventana de email: maximizada para que se vea el HTML completo

**Antes de grabar:**
1. Activa el flow en n8n (toggle ON)
2. Verifica que Ollama esté corriendo: `docker ps` → ver contenedor ollama
3. Ten el email abierto en otra pestaña/ventana lista para mostrar
4. Ejecuta una vez en seco para confirmar que todo funciona

**Edición:**
- Acelera al 1.5x los momentos de espera de Ollama (5-15 seg)
- Añade subtítulos con los KPIs principales sobre el email final
- Música de fondo opcional: lo-fi a volumen bajo

---

## Puntos clave a destacar en el demo

1. **Drive dinámico**: el flow no tiene un ID de archivo fijo — siempre toma el más reciente
2. **Parser COP**: maneja el formato `$ 188.571,00` correctamente (punto=miles, coma=decimal)
3. **IA local**: Ollama genera la narrativa sin consumir API externa ni incurrir en costos
4. **Email HTML profesional**: no es texto plano — es un reporte visual con tablas y cards
5. **Cero intervención**: una vez activo, corre solo cada lunes sin acción manual

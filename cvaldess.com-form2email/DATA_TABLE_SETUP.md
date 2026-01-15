# 📊 Configuración de Data Table para Rechazos de Formularios

## Paso 1: Crear la Data Table en n8n

### Opción A: Desde la Interfaz de n8n

1. Ve a tu instancia de n8n: `https://n8n.cvaldess.com/`
2. En el menú lateral, haz clic en **"Data Tables"** (o "Tablas de Datos")
3. Haz clic en **"Create Data Table"** (Crear Nueva Tabla)
4. Nombre de la tabla: `form_rejections`
5. Agrega los siguientes campos:

| Nombre del Campo | Tipo | Descripción |
|------------------|------|-------------|
| `timestamp` | DateTime | Fecha y hora del rechazo |
| `name` | String (100) | Nombre del formulario |
| `email` | String (100) | Email del formulario |
| `subject` | String (200) | Asunto del mensaje |
| `message` | Text | Mensaje completo |
| `rejection_reason` | Text | Razón detallada del rechazo |
| `rejection_type` | String (50) | Tipo: "PRE_VALIDATION" o "AI_VALIDATION" |
| `ip_address` | String (50) | Dirección IP del cliente |
| `user_agent` | Text | User Agent del navegador |

6. Guarda la tabla

### Opción B: Usando SQL (si tienes acceso directo)

```sql
CREATE TABLE form_rejections (
    id SERIAL PRIMARY KEY,
    timestamp TIMESTAMP NOT NULL DEFAULT NOW(),
    name VARCHAR(100),
    email VARCHAR(100),
    subject VARCHAR(200),
    message TEXT,
    rejection_reason TEXT,
    rejection_type VARCHAR(50),
    ip_address VARCHAR(50),
    user_agent TEXT
);

-- Índices para mejorar el rendimiento
CREATE INDEX idx_rejection_timestamp ON form_rejections(timestamp DESC);
CREATE INDEX idx_rejection_type ON form_rejections(rejection_type);
CREATE INDEX idx_rejection_email ON form_rejections(email);
```

---

## Paso 2: Modificar el Workflow

### Nuevos Nodos a Agregar

Vas a agregar **4 nodos nuevos**:

#### 1. **Log Pre-validation Reject** (Code Node)
- **Tipo:** Code (JavaScript)
- **Posición:** Entre "IF Pre-validation OK" (salida NO) y "Respond Pre-validation Reject"

**Código JavaScript:**
```javascript
// Preparar datos para la data table
const webhookData = $('Webhook').item.json;
const sanitized = $json.sanitized || { name: '', email: '', subject: '', message: '' };

return {
  json: {
    timestamp: new Date().toISOString(),
    name: sanitized.name || 'N/A',
    email: sanitized.email || 'N/A',
    subject: sanitized.subject || 'N/A',
    message: sanitized.message || 'N/A',
    rejection_reason: $json.reason || 'Sin razón especificada',
    rejection_type: 'PRE_VALIDATION',
    ip_address: webhookData.headers['cf-connecting-ip'] || webhookData.headers['x-forwarded-for'] || webhookData.headers['x-real-ip'] || 'Unknown',
    user_agent: webhookData.headers['user-agent'] || 'Unknown'
  }
};
```

#### 2. **Save to Data Table (Pre-validation)** (Data Table Node)
- **Tipo:** Data Table Store
- **Operación:** Insert
- **Data Table:** `form_rejections`
- **Configuración:**
  - Auto-map fields: Activado
  - Los campos se mapearán automáticamente desde el nodo anterior

#### 3. **Log AI Reject** (Code Node)
- **Tipo:** Code (JavaScript)
- **Posición:** Entre "IF AI Valid" (salida NO) y "Respond AI Reject"

**Código JavaScript:**
```javascript
// Preparar datos para la data table
const webhookData = $('Webhook').item.json;
const sanitized = $('Parse AI Response').item.json.sanitizedData || { name: '', email: '', subject: '', message: '' };

return {
  json: {
    timestamp: new Date().toISOString(),
    name: sanitized.name || 'N/A',
    email: sanitized.email || 'N/A',
    subject: sanitized.subject || 'N/A',
    message: sanitized.message || 'N/A',
    rejection_reason: $('Parse AI Response').item.json.reason || 'Sin razón especificada',
    rejection_type: 'AI_VALIDATION',
    ip_address: webhookData.headers['cf-connecting-ip'] || webhookData.headers['x-forwarded-for'] || webhookData.headers['x-real-ip'] || 'Unknown',
    user_agent: webhookData.headers['user-agent'] || 'Unknown'
  }
};
```

#### 4. **Save to Data Table (AI)** (Data Table Node)
- **Tipo:** Data Table Store
- **Operación:** Insert
- **Data Table:** `form_rejections`
- **Configuración:**
  - Auto-map fields: Activado

---

## Paso 3: Reconectar el Flujo

### Conexiones Originales a MODIFICAR:

**ANTES:**
```
IF Pre-validation OK → [NO] → Respond Pre-validation Reject
IF AI Valid → [NO] → Respond AI Reject
```

**DESPUÉS:**
```
IF Pre-validation OK → [NO] → Log Pre-validation Reject → 
    Save to Data Table (Pre-validation) → Respond Pre-validation Reject

IF AI Valid → [NO] → Log AI Reject → 
    Save to Data Table (AI) → Respond AI Reject
```

### Pasos Detallados en n8n:

1. **Abre el workflow** en el editor de n8n
2. **Desconecta** la salida NO de "IF Pre-validation OK" de "Respond Pre-validation Reject"
3. **Agrega** el nodo "Log Pre-validation Reject"
4. **Conecta:** "IF Pre-validation OK" [NO] → "Log Pre-validation Reject"
5. **Agrega** el nodo "Save to Data Table (Pre-validation)"
6. **Conecta:** "Log Pre-validation Reject" → "Save to Data Table (Pre-validation)"
7. **Conecta:** "Save to Data Table (Pre-validation)" → "Respond Pre-validation Reject"

8. **Desconecta** la salida NO de "IF AI Valid" de "Respond AI Reject"
9. **Agrega** el nodo "Log AI Reject"
10. **Conecta:** "IF AI Valid" [NO] → "Log AI Reject"
11. **Agrega** el nodo "Save to Data Table (AI)"
12. **Conecta:** "Log AI Reject" → "Save to Data Table (AI)"
13. **Conecta:** "Save to Data Table (AI)" → "Respond AI Reject"

---

## Paso 4: Probar el Workflow

### Enviar un Formulario Inválido (Pre-validación)

```bash
curl -X POST https://n8n.cvaldess.com/webhook/form-validator-secure \
  -H "Content-Type: application/json" \
  -d '{
    "name": "test",
    "email": "invalid-email",
    "subject": "abc",
    "message": "muy corto"
  }'
```

**Resultado Esperado:**
- ❌ Respuesta 400 con error
- ✅ Registro guardado en data table con `rejection_type: "PRE_VALIDATION"`

### Enviar un Formulario que Falle IA

```bash
curl -X POST https://n8n.cvaldess.com/webhook/form-validator-secure \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Robot Spammer",
    "email": "spam@test.com",
    "subject": "CLICK HERE WIN MONEY!!!!!",
    "message": "Visit http://bit.ly/scam to claim your prize! FREE MONEY GUARANTEED!!!"
  }'
```

**Resultado Esperado:**
- ❌ Respuesta 400 con error de IA
- ✅ Registro guardado en data table con `rejection_type: "AI_VALIDATION"`

---

## Paso 5: Verificar los Datos

### Ver los rechazos en la Data Table

1. Ve a **Data Tables** en n8n
2. Abre la tabla `form_rejections`
3. Deberías ver los registros de los formularios rechazados

### Consulta SQL (si tienes acceso)

```sql
-- Ver todos los rechazos
SELECT * FROM form_rejections ORDER BY timestamp DESC LIMIT 100;

-- Contar rechazos por tipo
SELECT rejection_type, COUNT(*) as total
FROM form_rejections
GROUP BY rejection_type;

-- Ver rechazos recientes (últimas 24 horas)
SELECT *
FROM form_rejections
WHERE timestamp > NOW() - INTERVAL '24 hours'
ORDER BY timestamp DESC;

-- Top 10 razones de rechazo
SELECT rejection_reason, COUNT(*) as count
FROM form_rejections
GROUP BY rejection_reason
ORDER BY count DESC
LIMIT 10;

-- Rechazos por IP
SELECT ip_address, COUNT(*) as attempts
FROM form_rejections
GROUP BY ip_address
HAVING COUNT(*) > 5
ORDER BY attempts DESC;
```

---

## Arquitectura Actualizada del Workflow

```
Webhook (POST)
    ↓
Sanitize & Pre-validate
    ↓
IF Pre-validation OK?
    ├─ NO → Log Pre-validation Reject 
    │          ↓
    │       Save to Data Table (Pre-validation)
    │          ↓
    │       Respond Pre-validation Reject (400)
    │
    └─ YES → AI Agent
                ↓
             Parse AI Response
                ↓
             IF AI Valid?
                ├─ NO → Log AI Reject
                │          ↓
                │       Save to Data Table (AI)
                │          ↓
                │       Respond AI Reject (400)
                │
                └─ YES → Send to Admin
                            ↓
                         Send to User
                            ↓
                         Respond Success (200)
```

---

## Beneficios del Sistema de Logging

### 📊 Análisis de Seguridad
- Identificar patrones de ataques
- Detectar IPs maliciosas
- Analizar tipos de intentos de spam

### 📈 Métricas
- Tasa de rechazo por tipo
- Razones más comunes de rechazo
- Horarios pico de intentos maliciosos

### 🛡️ Mejora Continua
- Identificar falsos positivos
- Ajustar reglas de validación
- Entrenar mejor el modelo de IA

### 🔍 Auditoría
- Registro completo de todos los intentos rechazados
- Trazabilidad de IPs problemáticas
- Evidencia para reportes de abuso

---

## Mantenimiento

### Limpieza Periódica

```sql
-- Eliminar registros mayores a 90 días
DELETE FROM form_rejections 
WHERE timestamp < NOW() - INTERVAL '90 days';

-- Archivar datos antiguos (opcional)
CREATE TABLE form_rejections_archive AS
SELECT * FROM form_rejections
WHERE timestamp < NOW() - INTERVAL '90 days';
```

### Monitoreo

```sql
-- Dashboard de rechazos diarios
SELECT 
    DATE(timestamp) as date,
    rejection_type,
    COUNT(*) as total
FROM form_rejections
WHERE timestamp > NOW() - INTERVAL '30 days'
GROUP BY DATE(timestamp), rejection_type
ORDER BY date DESC, rejection_type;
```

---

## Troubleshooting

### Error: "Data Table not found"
**Solución:** Verifica que creaste la data table con el nombre exacto `form_rejections`

### Error: "Field mismatch"
**Solución:** Verifica que todos los campos de la data table coincidan con los nombres en el código JavaScript

### Los datos no se guardan
**Solución:** 
1. Revisa los logs del workflow en n8n
2. Verifica que los nodos estén correctamente conectados
3. Prueba ejecutar el workflow manualmente paso por paso

### Performance lento
**Solución:**
1. Agrega índices a la tabla (ver SQL arriba)
2. Limpia registros antiguos periódicamente
3. Considera archivar datos históricos

---

*Última actualización: 15 de enero de 2026*
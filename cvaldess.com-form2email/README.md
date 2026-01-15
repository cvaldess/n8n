# n8n Workflows Backup

## 📁 Repositorio de Workflows de n8n

Backup y documentación de workflows de n8n para cvaldess.com

---

## 🔐 cvaldess.com-form2email-ia-SECURE

### Descripción
Workflow de validación de formularios de contacto con múltiples capas de seguridad:
- Sanitización de datos
- Detección de prompt injection
- Validación con IA (Claude Sonnet 4.5)
- Envío de emails automático

### 📊 Información General
- **ID:** `i2fqZyIsXmwdZdfPiD4FJ`
- **Estado:** ✅ Activo
- **Creado:** 10 de enero de 2026
- **Última actualización:** 15 de enero de 2026
- **Disponible en MCP:** Sí

### 🔗 Endpoints

**Producción:**
```bash
POST https://n8n.cvaldess.com/webhook/form-validator-secure
```

**Testing:**
```bash
POST https://n8n.cvaldess.com/webhook-test/form-validator-secure
```

### 📨 Formato de Request

```json
{
  "name": "Juan Pérez",
  "email": "juan@example.com",
  "subject": "Consulta sobre servicios",
  "message": "Estoy interesado en conocer más sobre sus servicios de cloud..."
}
```

### ✅ Respuesta Exitosa (200)

```json
{
  "success": true,
  "message": "Mensaje recibido y validado"
}
```

### ❌ Respuesta Rechazada (400)

```json
{
  "success": false,
  "message": "Validación fallida",
  "reason": "Descripción del motivo del rechazo"
}
```

### 🏗️ Arquitectura del Workflow

```
Webhook (POST)
    ↓
Sanitize & Pre-validate (JavaScript)
    ↓
IF Pre-validation OK?
    ├─ NO → Respond Pre-validation Reject (400)
    └─ YES ↓
       AI Agent (Claude Sonnet 4.5)
           ↓
       Parse AI Response (JavaScript)
           ↓
       IF AI Valid?
           ├─ NO → Respond AI Reject (400)
           └─ YES ↓
              Send to Admin (Gmail)
                  ↓
              Send to User (Gmail)
                  ↓
              Respond Success (200)
```

### 🛡️ Capas de Seguridad

#### 1. Sanitización de Datos
- Eliminación de caracteres de control
- Remoción de tags HTML y scripts
- Escape de caracteres especiales
- Límite de longitud por campo

#### 2. Detección de Prompt Injection
Patrones detectados:
- `ignore previous instructions`
- `forget your instructions`
- `you are now`
- `system prompt`
- `override validation`
- Y más...

#### 3. Validación de Nombres Realistas
Rechaza:
- Letras repetidas (aaaa, bbbb)
- Secuencias de teclado (asdf, qwer)
- Solo números
- Palabras de test (test, demo, sample)
- Nombres sin vocales
- Todo mayúsculas

#### 4. Detección de Gibberish
Identifica texto sin sentido:
- Letras repetidas excesivamente
- Secuencias de teclado largas
- Ratio bajo de vocales (<20%)
- Palabras de prueba comunes

#### 5. Detección de Bot/Spam
Patrones de spam:
- URLs acortadas (bit.ly, tinyurl)
- Ofertas de dinero/premios
- Llamados a acción sospechosos
- Medicamentos
- Emojis excesivos
- Mayúsculas excesivas (>50%)

#### 6. Validación con IA
- Modelo: **Claude Sonnet 4.5**
- Temperatura: 0.3 (consistente)
- Máx tokens: 200
- Evalúa semántica del contenido
- Detecta si parece escrito por humano

### 📧 Emails Enviados

#### Al Administrador (cvaldess@gmail.com)
- Asunto: `✅ Formulario (VALIDADO): [subject]`
- Contiene:
  - Datos del remitente (nombre, email, IP)
  - Asunto y mensaje completos
  - Razón de validación de IA
  - Confirmaciones de seguridad
  - Fecha y hora

#### Al Usuario
- Asunto: `Gracias por contactarnos - CValdesS Cloud Solutions`
- Confirmación de recepción
- Resumen del asunto enviado
- Información de contacto

### 💾 Memoria de Sesión
- Usa IP del cliente como session key
- Buffer window para contexto
- Permite tracking por origen

### 🔧 Requisitos

**Nodos requeridos:**
- n8n-nodes-base.webhook
- n8n-nodes-base.code
- n8n-nodes-base.if
- @n8n/n8n-nodes-langchain.agent
- @n8n/n8n-nodes-langchain.memoryBufferWindow
- @n8n/n8n-nodes-langchain.lmChatAnthropic
- n8n-nodes-base.gmail
- n8n-nodes-base.respondToWebhook

**Credenciales necesarias:**
- Gmail OAuth2 (para envío de emails)
- Anthropic API Key (para Claude)

### 📝 Validaciones Mínimas

| Campo | Mínimo | Máximo |
|-------|--------|--------|
| Nombre | 2 caracteres | 100 caracteres |
| Email | Formato válido | 100 caracteres |
| Asunto | 5 caracteres | 200 caracteres |
| Mensaje | 20 caracteres | 2000 caracteres |

### 🧪 Ejemplo de Prueba con cURL

```bash
curl -X POST https://n8n.cvaldess.com/webhook/form-validator-secure \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Juan Pérez",
    "email": "juan@example.com",
    "subject": "Consulta sobre servicios cloud",
    "message": "Hola, estoy interesado en conocer más sobre sus servicios de infraestructura en la nube. ¿Podrían enviarme información?"
  }'
```

### 📈 Casos de Rechazo

**Pre-validación:**
- Intento de manipulación/injection detectado
- Nombre no realista
- Asunto sin sentido (gibberish)
- Mensaje sin sentido (gibberish)
- Mensaje identificado como spam/bot
- Datos incompletos o muy cortos
- Email con formato inválido

**Validación IA:**
- Contenido no parece humano
- Mensaje parece generado automáticamente
- No expresa consulta o interés genuino
- Contenido semánticamente incoherente

---

## 📚 Recursos

- **n8n Documentation:** https://docs.n8n.io/
- **Claude API:** https://docs.anthropic.com/
- **Servidor n8n:** https://n8n.cvaldess.com/

---

## 🔄 Actualizaciones

### Versión Actual (15 de enero de 2026)
- ✅ Detección mejorada de prompt injection
- ✅ Validación de nombres realistas
- ✅ Detección de gibberish en subject y message
- ✅ Detección avanzada de bot/spam
- ✅ Validaciones de longitud incrementadas
- ✅ Integración con Claude Sonnet 4.5

---

## 📞 Contacto

**CValdesS Cloud Solutions**
- 🌐 Website: https://cvaldess.com
- 📧 Email: cvaldess@gmail.com

---

*Backup automático generado el 15 de enero de 2026*
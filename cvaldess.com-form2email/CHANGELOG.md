# Changelog - cvaldess.com-form2email-ia-SECURE

Todos los cambios notables a este workflow serán documentados en este archivo.

---

## [v2.0.0] - 2026-01-15

### 🆕 Agregado
- **Data Table Integration:** Sistema completo de registro de formularios rechazados
  - Tabla `form_rejections` para almacenar todos los rechazos
  - Logging de rechazos en pre-validación
  - Logging de rechazos en validación de IA
  - Captura de IP del cliente
  - Captura de User Agent del navegador
  - Timestamp automático para cada rechazo

### 📊 Nuevos Nodos
1. **Log Pre-validation Reject** (Code)
   - Prepara datos de rechazos de pre-validación
   - Extrae IP del cliente de headers
   - Formatea datos para data table
   
2. **Save to Data Table (Pre-validation)** (Data Table Store)
   - Inserta registros en `form_rejections`
   - Marca tipo como "PRE_VALIDATION"
   
3. **Log AI Reject** (Code)
   - Prepara datos de rechazos de IA
   - Incluye razón específica de Claude
   - Extrae metadata del request
   
4. **Save to Data Table (AI)** (Data Table Store)
   - Inserta registros en `form_rejections`
   - Marca tipo como "AI_VALIDATION"

### 🔄 Modificado
- **IF Pre-validation OK:** Agregada ruta a logging antes de responder rechazo
- **IF AI Valid:** Agregada ruta a logging antes de responder rechazo
- **Flujo de rechazos:** Ahora incluye persistencia antes de responder al cliente

### 📈 Beneficios
- Análisis de patrones de ataque
- Identificación de IPs maliciosas
- Métricas de tasa de rechazo
- Auditoría completa de intentos fallidos
- Base para mejora continua del sistema

### 📝 Documentación
- Agregado `DATA_TABLE_SETUP.md` con instrucciones completas
- Esquema SQL para crear la tabla
- Queries de ejemplo para análisis
- Guía de troubleshooting
- Instrucciones de mantenimiento

---

## [v1.0.0] - 2026-01-10

### 🆕 Inicial
- **Webhook:** Endpoint POST para recibir formularios
- **Sanitización:** Limpieza avanzada de datos de entrada
- **Pre-validación:** Detección de:
  - Prompt injection
  - Nombres no realistas
  - Gibberish en subject y message
  - Mensajes de bot/spam
  - Validaciones básicas de longitud
  
- **Validación con IA:** 
  - Integración con Claude Sonnet 4.5
  - Análisis semántico del contenido
  - Detección de contenido no humano
  
- **Notificaciones:**
  - Email al administrador (cvaldess@gmail.com)
  - Email de confirmación al usuario
  
- **Respuestas HTTP:**
  - 200 para mensajes válidos
  - 400 para rechazos (pre-validación y IA)

### 🛡️ Seguridad
- Sanitización de caracteres de control
- Remoción de HTML y scripts
- Escape de caracteres especiales
- Límites de longitud por campo
- Detección de patrones maliciosos
- Validación multi-capa

---

## Formato de Versiones

El proyecto sigue [Semantic Versioning](https://semver.org/):
- **MAJOR:** Cambios incompatibles en la API
- **MINOR:** Nueva funcionalidad compatible con versiones anteriores
- **PATCH:** Correcciones de bugs compatibles

---

## Tipos de Cambios

- `🆕 Agregado` - Nueva funcionalidad
- `🔄 Modificado` - Cambios en funcionalidad existente
- `🗑️ Eliminado` - Funcionalidad removida
- `🐛 Corregido` - Corrección de bugs
- `🛡️ Seguridad` - Mejoras de seguridad
- `📊 Data` - Cambios relacionados con datos
- `📝 Documentación` - Solo cambios en documentación
- `🎨 Estilo` - Cambios de formato/estilo
- `♻️ Refactor` - Cambios de código sin afectar funcionalidad
- `⚡ Performance` - Mejoras de rendimiento
- `✅ Tests` - Agregado o corrección de tests

---

*Última actualización: 15 de enero de 2026*
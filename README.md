# 📦 Repositorio de Workflows n8n

Backup y documentación de workflows de n8n para **cvaldess.com**

---

## 📂 Estructura del Repositorio

```
n8n/
├── cvaldess.com-form2email/
│   ├── cvaldess.com-form2email-ia-SECURE.json
│   └── README.md
└── README.md (este archivo)
```

---

## 🔧 Workflows Disponibles

### 1. [cvaldess.com-form2email-ia-SECURE](./cvaldess.com-form2email/)

**Estado:** ✅ Activo  
**Descripción:** Workflow de validación de formularios con IA y múltiples capas de seguridad

**Características principales:**
- 🛡️ Sanitización avanzada de datos
- 🤖 Validación con Claude Sonnet 4.5
- 🔒 Detección de prompt injection
- 📧 Envío automático de emails
- 🚫 Protección anti-spam y anti-bot

**Endpoints:**
- Producción: `https://n8n.cvaldess.com/webhook/form-validator-secure`
- Testing: `https://n8n.cvaldess.com/webhook-test/form-validator-secure`

**[📖 Ver documentación completa](./cvaldess.com-form2email/README.md)**

---

## 🚀 Cómo Usar Este Repositorio

### Importar un Workflow

1. Ve a tu instancia de n8n
2. Haz clic en "Import from File"
3. Selecciona el archivo `.json` del workflow deseado
4. Configura las credenciales necesarias
5. Activa el workflow

### Requisitos Generales

- **n8n** v1.0.0 o superior
- **Node.js** v18 o superior
- Acceso a las APIs/servicios utilizados por cada workflow

---

## 📊 Estadísticas

| Workflow | Archivos | Última Actualización | Estado |
|----------|----------|---------------------|--------|
| cvaldess.com-form2email | 2 | 15 de enero de 2026 | ✅ Activo |

---

## 🔐 Seguridad

Todos los workflows en este repositorio han sido diseñados con seguridad en mente:

- ✅ Validación de entrada de datos
- ✅ Sanitización de contenido
- ✅ Protección contra inyecciones
- ✅ Detección de spam y bots
- ✅ Manejo seguro de credenciales

---

## 📚 Recursos

- **n8n Oficial:** https://n8n.io/
- **Documentación n8n:** https://docs.n8n.io/
- **Servidor n8n:** https://n8n.cvaldess.com/
- **Website:** https://cvaldess.com/

---

## 📝 Convenciones

### Nomenclatura de Workflows
```
[dominio]-[función]-[versión]
```

Ejemplos:
- `cvaldess.com-form2email-ia-SECURE`
- `cvaldess.com-newsletter-automation`

### Estructura de Carpetas
Cada workflow tiene su propia carpeta que contiene:
- `[nombre-workflow].json` - Definición del workflow
- `README.md` - Documentación específica

---

## 🤝 Contribuciones

Este repositorio es privado y está mantenido por el equipo de CValdesS Cloud Solutions.

---

## 📞 Contacto

**CValdesS Cloud Solutions**
- 🌐 Website: https://cvaldess.com
- 📧 Email: cvaldess@gmail.com
- 🔗 GitHub: [@cvaldess](https://github.com/cvaldess)

---

## 📜 Licencia

© 2026 CValdesS Cloud Solutions. Todos los derechos reservados.

---

*Última actualización: 15 de enero de 2026*
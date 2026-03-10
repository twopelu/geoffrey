# Especificación Técnica: Agente de Comunicación de Skills Multimodales vía Interfaz Web (Tipo Chat)

## 1. Descripción General

El sistema consiste en un **agente conversacional accesible desde una interfaz web tipo chat**. Este agente será capaz de:

- Responder y comunicarse con el usuario en lenguaje natural.
- Incorporar el uso de distintas **skills** (capacidades funcionales) que pueden ser extendidas o añadidas modularmente.
- Soportar múltiples modelos de IA, incluyendo versiones gratuitas como ChatGPT, para procesar y generar respuestas.
- Tener una arquitectura escalable que permita la integración posterior de otras funciones o modelos.

## 2. Requisitos Funcionales

### 2.1 Chat Web

- Interfaz web básica tipo chat, para mensajes usuario ↔ agente.
- Mostrar histórico de conversación.
- Permitir enviar textos (opcional: archivos e imágenes para skills futuras).
- Indicar visualmente cuándo el agente está procesando una respuesta.

### 2.2 Skills Modulares

- El agente puede invocar **skills** según el contexto de la conversación o comandos explícitos.
- Cada skill implementa una funcionalidad específica (ej: calculadora, búsqueda web, API externas).
- Los skills deben ser discoverables y gestionables (encender/apagar, añadir, actualizar).
- API interna clara para la definición de nuevas skills.

### 2.3 Selección de Modelo IA

- Integración con servicios gratuitos de IA (ej: OpenAI GPT-3.5-Turbo, HuggingFace, etc).
- Configuración para seleccionar el modelo IA predeterminado o permitir al usuario seleccionar modelo por sesión o mensaje.
- Fallback automático a modelos gratuitos si un modelo premium no está disponible.

### 2.4 Orquestación de Skills y Modelos

- Sistema de routing para decidir a qué modelo o skill enviar el input del usuario.
- Capacidad de combinar respuestas del modelo IA y de las skills si es pertinente.
- Facilidad para extraer comandos de skill de la entrada del usuario (parseo de prompt, comandos tipo `/skill`).

## 3. Requisitos No Funcionales

- **Extensible:** Arquitectura basada en plugins o módulos para skills.
- **Portabilidad:** Frontend compatible con navegadores modernos y fácil despliegue.
- **Simplicidad:** Backend ligero, preferiblemente sin dependencias complejas.
- **Mantenibilidad:** Código comentado y modular para facilitar mejoras futuras.

## 4. Arquitectura de Alto Nivel

```
[Usuario]
   ↓
[Frontend Web Chat] ←→ [Backend API Gateway] ←→ [Orquestador]
                                                     │
                  +-------------------------+--------+----------+
                  |                         |                   |
           [Modelo IA 1: ChatGPT 3.5]   [Skill: Calculadora]    [Skill: API externa]
                  |                         |                   |
          [Otros modelos IA]          [Skills futuras]     [Skills futuras]
```

## 5. Interfaces y API (Borrador)

### 5.1 API Skill

```javascript
// Estructura básica de una skill
module.exports = {
  name: 'nombre_skill',
  description: 'Descripción funcional',
  // Función que recibe input y contexto, retorna respuesta
  handler: async (input, context) => { ... }
};
```

### 5.2 Llamada desde el backend para cada mensaje

```javascript
POST /api/chat
{
  "session_id": "...",
  "user_message": "...",
  "selected_ai_model": "gpt-3.5-turbo",
  // Opcional: Forzar skill
  "force_skill": "calculadora"
}
```

### 5.3 Respuesta

```javascript
{
  "messages": [
    {"sender":"user", "text":"¿Cuál es la raíz cuadrada de 16?"},
    {"sender":"agent", "text":"La raíz cuadrada de 16 es 4.", "source":"calculadora"}
  ],
  "skill_used": "calculadora",
  "ai_model_used": "gpt-3.5-turbo"
}
```

## 6. Consideraciones de Seguridad y Privacidad

- No almacenar mensajes sensibles a menos que sea absolutamente necesario.
- Permitir eliminación del histórico por parte del usuario.
- Validar y sanitizar inputs del usuario antes de procesarlos.
- Usar variables de entorno para almacenar claves API.

## 7. Futuras Extensiones

- Soporte para skills con entrada multimodal (voz, imagen).
- Soporte para sesión multiusuario.
- Integración con APIs de autenticación para personalizar skills.
- Análisis de sentimiento y personalización de respuestas.
- Persistencia de sesiones y exportación de conversaciones.

# pf_ia_automation
# Ecosistema de Automatización IA para la Gestión de Tickets de Soporte

## 1. Descripción del proyecto

Este proyecto consiste en el desarrollo de un ecosistema de automatización inteligente orientado a la gestión de consultas y solicitudes de clientes mediante correo electrónico.

El sistema utiliza **n8n** como plataforma de automatización, **Google Gemini 2.5 Flash** para el análisis y clasificación de mensajes, **Notion** como base de datos y **Gmail** como canal de comunicación.

A través de dos workflows, el ecosistema permite recibir consultas, procesar su contenido mediante inteligencia artificial, registrar tickets, clasificarlos según su categoría y derivarlos al área correspondiente. También incorpora un mecanismo de intervención humana (Human in the Loop) para las solicitudes de facturación que requieren aprobación antes de continuar su procesamiento.

## 2. Objetivos del proyecto

El objetivo principal es automatizar tareas administrativas relacionadas con la recepción y gestión inicial de consultas de clientes.

Objetivos específicos:

- Automatizar la recepción de correos electrónicos.
- Utilizar inteligencia artificial para clasificar y analizar consultas.
- Identificar la categoría y el sentimiento de cada mensaje.
- Generar resúmenes automáticos de las solicitudes.
- Centralizar la información en una base de datos de Notion.
- Derivar las consultas según su categoría.
- Incorporar aprobación humana en los procesos que lo requieren.
- Registrar información relacionada con errores y generar notificaciones.

## 3. Tecnologías utilizadas

| Tecnología | Función dentro del proyecto |
|---|---|
| n8n Self Hosted | Orquestación y ejecución de workflows. |
| Google Gemini 2.5 Flash | Análisis y clasificación de consultas mediante IA. |
| Notion | Almacenamiento y gestión de tickets y registros de errores. |
| Gmail | Recepción de consultas y envío de notificaciones. |
| JavaScript | Procesamiento, interpretación y normalización de respuestas JSON. |

**Nota sobre el modelo de IA:** se utilizó Google Gemini 2.5 Flash debido a la disponibilidad de acceso y créditos para el desarrollo del proyecto. Esta elección constituye una adaptación respecto de la consigna original, que proponía OpenAI GPT o Anthropic Claude.

## 4. Arquitectura general del sistema

El ecosistema está compuesto por dos workflows independientes:

1. **Workflow principal: Gestión automatizada de tickets.** Se encarga de recibir, analizar, clasificar, registrar y derivar las consultas.
2. **Workflow secundario: Gestión de errores.** Está diseñado para registrar información de errores de ejecución en Notion y enviar notificaciones mediante Gmail.

### 4.1. Workflow principal: Gestión automatizada de tickets

El proceso comienza cuando Gmail detecta la llegada de un nuevo correo electrónico.

A continuación, se ejecutan las siguientes etapas:

1. **Gmail Trigger:** detecta nuevos correos electrónicos.
2. **Gmail - Get a Message:** obtiene el contenido del mensaje recibido.
3. **Notion - Create Database Page:** crea un nuevo registro en la base de datos SOPORTE TICKETS.
4. **Gemini 2.5 Flash:** analiza el contenido del correo y genera una clasificación estructurada.
5. **Code in JavaScript:** interpreta la respuesta JSON de Gemini y normaliza los datos obtenidos.
6. **Notion - Update Database Page:** actualiza el ticket con la información procesada.
7. **Switch:** evalúa la categoría asignada y deriva el ticket hacia una de las cuatro rutas disponibles.

### 4.2. Clasificación de consultas

El sistema contempla cuatro categorías:

| Categoría | Acción automatizada |
|---|---|
| Error Técnico | Actualiza el estado del ticket a "Derivado a Soporte" y envía una notificación al equipo correspondiente. |
| Ventas | Actualiza el estado a "Derivado a Ventas" y notifica al área comercial. |
| Facturación | Actualiza el ticket a "En curso" y solicita aprobación humana mediante Gmail. |
| Otro | Actualiza el estado a "Pendiente revisión manual" y notifica al administrador. |

Además de la categoría, Gemini genera dos datos complementarios:

- **Sentimiento:** positivo, negativo o neutro.
- **Resumen:** síntesis de la consulta con un máximo de 15 palabras.

La respuesta del modelo se estructura en formato JSON para facilitar su procesamiento dentro de n8n.

### 4.3. Intervención humana (Human in the Loop)

La rama de Facturación incorpora un mecanismo de aprobación humana antes de continuar con el procesamiento.

Su funcionamiento es el siguiente:

1. El sistema registra el ticket con estado "En curso".
2. El campo "Aprobado" se establece inicialmente en falso.
3. Gmail envía una notificación con un enlace de aprobación.
4. El workflow queda detenido en el nodo Wait.
5. Cuando la persona accede al enlace de reanudación, el workflow continúa.
6. Notion actualiza el estado del ticket a "Listo" y el campo "Aprobado" pasa a verdadero.

Este mecanismo permite incorporar supervisión humana en una etapa del proceso que requiere validación.

**Consideración:** la implementación utiliza el enlace de reanudación como mecanismo de aprobación. No contempla una ruta diferenciada de rechazo ni una validación adicional de identidad del usuario que accede al enlace.

## 5. Bases de datos en Notion

El proyecto utiliza dos bases de datos para organizar la información.

### 5.1. SOPORTE TICKETS

Es la base principal donde se almacenan las consultas recibidas y su información procesada.

Propiedades principales:

- Asunto.
- Remitente.
- Cuerpo del mensaje.
- Categoría.
- Sentimiento.
- Resumen.
- Estado.
- Fecha de ingreso.
- Fecha de cierre.
- Aprobado.
- Notas.

Esta base permite centralizar las solicitudes y realizar un seguimiento de su clasificación y estado.

### 5.2. Errores_Log

Es la base destinada al registro de información relacionada con errores de ejecución.

Sus propiedades principales son:

- Fecha y hora del error.
- Nodo fallido.
- Mensaje de error.
- ID de ejecución.
- Ticket relacionado (opcional).

## 6. Workflow secundario: Gestión de errores

El segundo workflow fue diseñado para centralizar el registro y la notificación de errores de ejecución.

Está compuesto por los siguientes nodos:

1. **Error Trigger:** recibe información de errores de workflows.
2. **Notion - Create Database Page:** registra los datos del error en la base Errores_Log.
3. **Gmail - Send Message:** envía una notificación por correo electrónico.

**Estado de validación:** las acciones de creación de registros en Notion y envío de correos fueron probadas manualmente. Sin embargo, no se logró validar el disparo automático del workflow secundario ante un error generado desde el workflow principal. Por este motivo, dicha activación no se presenta como una funcionalidad comprobada.

## 7. Pruebas realizadas

Durante el desarrollo se realizaron pruebas sobre las distintas etapas del sistema.

Se verificaron los siguientes comportamientos:

- Recepción de correos electrónicos mediante Gmail.
- Obtención del contenido de los mensajes.
- Creación de registros en Notion.
- Clasificación de consultas mediante Gemini.
- Procesamiento de respuestas JSON con JavaScript.
- Derivación de consultas mediante el nodo Switch.
- Actualización de estados según la categoría.
- Envío de notificaciones mediante Gmail.
- Ejecución del mecanismo de aprobación humana en la rama Facturación.
- Reanudación del workflow y actualización del ticket después de la aprobación.
- Creación manual de registros de errores y envío manual de notificaciones.

La activación automática del workflow de gestión de errores permanece como una limitación pendiente de validación.

## 8. Diagramas de arquitectura

Se incluyen los diagramas visuales de ambos workflows para facilitar la comprensión de su estructura y funcionamiento.

### Workflow principal

Adjuntado en archivos como captura de pantalla

### Workflow de gestión de errores

Adjuntado en archivos como captura de pantalla

Los archivos editables originales también se encuentran disponibles en el repositorio.

## 9. Estructura del repositorio

El repositorio contiene la documentación, los diagramas, las exportaciones de n8n y las evidencias del proyecto.

```text
Ecosistema-Automatizacion-IA/
│
├── README.md
│
├── diagramas/
│   ├── Captura de pantalla workflow secundario
│   ├──Captura de pantalla workflow principal
│   ├──README.md
│   └── Workflow principal
│
└── video/
    └── demo_proyecto.mp4
```

## 10. Acceso a la documentación y demostración

**Base de datos de Notion:**

PRINCIPAL: https://app.notion.com/p/412f7b65cd63809b9bc8caf0cf549d7e?v=412f7b65cd63804e8003000ca81a061e&source=copy_link
MANEJO DE ERRORES: https://app.notion.com/p/413f7b65cd638036a2e4c8ff96b50cb3?v=413f7b65cd6380d88eb8000c8cccfd26&source=copy_link

**Video demostrativo:**

[Agregar aquí el enlace al video de demostración]

## 11. Consideraciones finales

El desarrollo permitió integrar herramientas de automatización, inteligencia artificial, bases de datos y comunicación para construir un sistema de gestión de consultas con distintos niveles de procesamiento y supervisión.

El proyecto demuestra cómo estas tecnologías pueden combinarse para reducir tareas administrativas repetitivas, mejorar la organización de la información y facilitar la derivación de solicitudes.

También permitió trabajar con estructuras JSON, expresiones dinámicas, procesamiento mediante JavaScript, integración de APIs y mecanismos de intervención humana dentro de workflows automatizados.

La solución constituye un prototipo funcional y una base para futuras mejoras, como la validación de la activación automática del workflow de errores, la incorporación de mecanismos de rechazo en el proceso de aprobación y el fortalecimiento de los controles de seguridad.


**Proyecto desarrollado como entrega final del curso de Automatización e Inteligencia Artificial.**

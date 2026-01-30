# AgendaBot (n8n + Telegram)

AgendaBot es un bot conversacional implementado en n8n y desplegado en Telegram para ayudar a las personas a organizar citas, tareas, recordatorios, hábitos y listas desde un chat. El flujo está diseñado como una máquina de estados con persistencia en Google Sheets, de modo que cada conversación puede continuar paso a paso sin perder contexto.

## Finalidad y objetivo

**Finalidad:** centralizar la organización personal (agenda, tareas y hábitos) directamente en Telegram, sin necesidad de apps adicionales.

**Objetivo:** ofrecer una experiencia guiada y simple donde el usuario responde con números y el bot se encarga de registrar, consultar, reprogramar o cancelar elementos, además de enviar recordatorios automáticos.

## ¿Cómo funciona?

1. **Entrada:** el flujo comienza con un `Telegram Trigger` que recibe cada mensaje.
2. **Control de acceso:** se valida si el usuario existe y si está permitido. Si no existe, se crea en la hoja `Usuarios` y se inicializa su estado en `estado_usuario`.
3. **Estado conversacional:** se guarda la `pantalla_actual`, `paso_actual` y datos temporales (fecha, hora, nombre, motivo, etc.) en Google Sheets para saber en qué parte del menú va el usuario.
4. **Menús y flujos:** el bot responde con menús numerados y valida la opción elegida. Cada opción dispara un subflujo (agenda, tareas, hábitos, listas, reportes, configuración y administración).
5. **Persistencia:** todas las entidades se guardan en Google Sheets (citas, tareas, hábitos, listas) con IDs, autor y estado.
6. **Recordatorios automáticos:** un flujo de recordatorios calcula la diferencia de tiempo y envía avisos de citas cuando faltan $x$ minutos, usando la zona horaria `America/Bogota`.

## Funcionalidades principales

- **Agenda (citas)**
	- Crear, consultar, reprogramar y cancelar citas.
	- Validación de fecha y hora en formato 24 horas.
	- Confirmaciones antes de guardar.

- **Tareas**
	- Crear, consultar, eliminar y marcar como completadas.
	- Priorización (alta, media, baja).
	- Resúmenes listados por el bot.

- **Hábitos**
	- Crear, consultar y eliminar hábitos.
	- Configuración de frecuencia y horario de recordatorio.

- **Recordatorios**
	- Activación/desactivación desde el menú de administrador.
	- Envío automático de recordatorios de citas según ventana de tiempo.

- **Listas**
	- Gestión de listas y sus ítems (por ejemplo compras o pendientes).

- **Reportes básicos**
	- Resumen de citas, tareas y hábitos registrados.

- **Configuración**
	- Ajustes de comportamiento básicos almacenados en la hoja `Configuracion`.

## Arquitectura técnica

- **Plataforma:** n8n (workflow JSON importable).
- **Canal:** Telegram Bot API.
- **Persistencia:** Google Sheets (modo “service account”).
- **Lógica adicional:** nodos de código JavaScript para:
	- Normalizar listas y conteos.
	- Construir mensajes dinámicos.
	- Validar opciones y seleccionar elementos.

## Modelo de datos (Google Sheets)

El workflow utiliza varias hojas en un mismo documento:

- `Usuarios`: registro de usuarios, rol, permisos y fecha de creación.
- `estado_usuario`: estado conversacional, pantalla actual, paso actual y datos temporales.
- `Citas`: agenda de citas por usuario.
- `Tareas`: tareas con prioridad, estado y fecha objetivo.
- `Habitos`: hábitos con frecuencia y hora de recordatorio.
- `Listas`: listas personalizadas.
- `Items_Lista`: ítems asociados a cada lista.
- `Configuracion`: parámetros operativos del bot (por ejemplo recordatorios).

## Flujo de interacción (resumen)

1. Usuario inicia conversación.
2. Bot muestra menú principal.
3. Usuario responde con un número.
4. Bot valida la opción y guía paso a paso.
5. Se guardan datos temporales y se confirma antes de persistir.
6. El usuario puede cancelar en cualquier momento con la opción indicada en el menú.

## Requisitos

- Instancia de n8n (self-hosted o cloud).
- Bot de Telegram creado con BotFather.
- Credenciales de **Google Sheets** (service account).
- Documento de Google Sheets con las hojas indicadas arriba.

## Instalación y configuración

1. Importa el archivo JSON del workflow en n8n.
2. Configura las credenciales de Telegram.
3. Configura las credenciales de Google Sheets (service account).
4. Actualiza las referencias al documento de Google Sheets si usas otro.
5. Activa el workflow.

## Uso básico

- Escribe en el chat del bot y responde con números del menú.
- El bot te guía para crear o consultar elementos.
- Usa la opción de cancelar para regresar al menú principal.

## Seguridad y control de acceso

El acceso se controla desde la hoja `Usuarios` mediante el campo `permitido`. Si está en `NO`, el bot deniega el acceso automáticamente. Esto permite administrar usuarios autorizados sin tocar el flujo.

## Autores

- Davisson Adriel
- Edgar Leal

---

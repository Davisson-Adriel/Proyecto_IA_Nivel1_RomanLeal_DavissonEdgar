# AgendaBot — Documento técnico de funcionamiento

Este documento profundiza en el funcionamiento interno del bot AgendaBot (n8n + Telegram), describiendo su arquitectura, flujo conversacional, persistencia y lógica operativa.

## 1) Resumen del sistema

AgendaBot es un bot conversacional en Telegram implementado como un workflow de n8n. Su lógica opera como una máquina de estados donde el “estado de conversación” se persiste en Google Sheets. Cada interacción del usuario puede continuar desde el punto exacto donde quedó (pantalla y paso actual).

Componentes principales:

- **Canal de entrada:** Telegram Bot API.
- **Orquestación:** n8n (workflow JSON importable).
- **Persistencia:** Google Sheets (service account).
- **Lógica auxiliar:** nodos de JavaScript para validaciones y mensajes dinámicos.

## 2) Arquitectura general

### 2.1 Flujo de entrada

1. **Telegram Trigger** recibe mensajes (updates tipo `message`).
2. **Validación de usuario** busca al emisor en la hoja `Usuarios`.
3. **Registro automático** si el usuario no existe.
4. **Control de permisos**: si `permitido = NO`, se envía un mensaje de acceso denegado.
5. **Carga de estado** desde `estado_usuario` (pantalla actual, paso, datos temporales).
6. **Router de pantallas**: el mensaje se enruta al subflujo correspondiente según el valor de `pantalla_actual`.

### 2.2 Persistencia y estado

La hoja `estado_usuario` conserva:

- `pantalla_actual`: pantalla activa (por ejemplo `MENU_SELECCION`, `MENU_TAREAS`, `NUEVA_CITA`, etc.).
- `paso_actual`: paso dentro de un flujo (por ejemplo, recolección de fecha, hora, nombre…).
- `timestamp_ultima_interacción`: para auditoría y control de inactividad.
- Contadores (`n_citas`, `n_tareas`, `n_habitos`).
- Campos temporales (`fecha_temporar`, `hora_temporal`, `nombre_temporal`, etc.) usados durante formularios.

Este diseño permite que el bot sea **tolerante a interrupciones**, ya que el usuario puede pausar y continuar el flujo sin perder información.

## 3) Máquina de estados (pantallas)

El bot se comporta como un conjunto de pantallas con transiciones determinísticas. La pantalla activa se actualiza en `estado_usuario` después de cada paso.

Ejemplos de pantallas:

- `MENU_SELECCION`: menú principal.
- `MENU_TAREAS`: menú de tareas.
- `NUEVA_CITA`: creación de citas (flujo por pasos).
- `ELIMINAR_TAREA`, `NUEVO_HABITO`, `LISTAS`, `REPORTES`, etc.

El nodo “router” evalúa `pantalla_actual` y envía el mensaje al bloque correcto.

## 4) Flujos funcionales (detalle)

### 4.1 Agenda (citas)

**Objetivo:** crear, consultar y reprogramar citas.

**Pasos típicos de creación:**

1. Solicitar fecha.
2. Validar formato.
3. Solicitar hora.
4. Validar formato (24h).
5. Solicitar nombre.
6. Solicitar motivo.
7. Solicitar canal.
8. Confirmación final.
9. Persistir en hoja `Citas`.

**Reprogramación:**

- Se listan citas del usuario.
- Se selecciona una cita por índice.
- Se actualiza fecha/hora y se confirma.

### 4.2 Tareas

**Acciones:**

- Crear nueva tarea (título, prioridad, fecha objetivo).
- Consultar tareas existentes.
- Eliminar tarea.
- Marcar como completada.

La prioridad se captura como opción numérica y se guarda junto con el estado.

### 4.3 Hábitos

**Acciones:**

- Registrar hábito con frecuencia y hora de recordatorio.
- Consultar hábitos.
- Eliminar hábitos.

Los hábitos incluyen: `id_habito`, `nombre`, `frecuencia`, `hora_recordatorio`, `estado`.

### 4.4 Listas

- Listas personalizadas (por ejemplo compras, pendientes).
- Ítems asociados en `Items_Lista`.

### 4.5 Recordatorios

Existen dos tipos principales:

1. **Recordatorio de citas**: calcula tiempo restante y envía aviso cuando el evento está dentro de la ventana configurada.
2. **Recordatorio general**: resumen de citas, tareas y hábitos (consolidado para el usuario).

## 5) Validaciones y manejo de errores

- **Opciones inválidas:** si el usuario escribe un número fuera del menú, el bot reenvía las opciones.
- **Formato de hora:** valida `HH:MM` en 24h.
- **Formato de fecha:** se verifica consistencia antes de guardar.
- **Cancelación:** el usuario puede cancelar y regresar al menú.

## 6) Lógica dinámica en JavaScript

El flujo incluye nodos de código para:

- Transformar datos de Sheets en listas navegables.
- Generar mensajes dinámicos con tablas/formatos.
- Validar opciones seleccionadas y evitar errores de índice.
- Calcular conteos y métricas para reportes.

## 7) Seguridad y control de acceso

El control de permisos se maneja desde `Usuarios`:

- `permitido = SI`: acceso normal.
- `permitido = NO`: acceso denegado.

Esto permite administrar usuarios sin modificar el workflow.

## 8) Configuración operativa

- La hoja `Configuracion` mantiene parámetros generales.
- El administrador puede activar/desactivar recordatorios desde el menú.

## 9) Consideraciones de despliegue

Requisitos:

- n8n instalado (self-hosted o cloud).
- Bot de Telegram configurado.
- Cuenta de servicio de Google con acceso al documento.

## 10) Extensibilidad

AgendaBot se puede ampliar de forma modular:

- Integración con Google Calendar.
- Envío de recordatorios por email.
- Notificaciones push externas.
- Panel web administrativo.

---

Autores:

- Davisson Adriel
- Edgar Leal

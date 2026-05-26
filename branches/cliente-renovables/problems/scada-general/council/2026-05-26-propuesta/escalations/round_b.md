# Escalations Round B — 2026-05-26-propuesta

## [arquetipo-esceptico-operacional]
Pregunta: ¿Existe un equipo claro responsable de incidencias de plataforma en planta (K8s, volúmenes, certificados)?
Respuesta del user: Difuso todavía — no está definido.

## [arquetipo-esceptico-operacional]
Pregunta: Si una release rompe ingestión/alarmas en todos los parques, ¿preferís parar despliegues o seguir con operación degradada?
Respuesta del user: Sistema de rollback.

## [arquetipo-esceptico-operacional]
Pregunta: ¿Qué ventana máxima de datos tardíos es aceptable antes de que dashboards/reportes/alarmas dejen de ser fiables?
Respuesta del user: Habrá que informar (al usuario/operario de que los datos tienen retraso).

## [arquitecto-scada-distribuido]
Pregunta: ¿En cuánto tiempo debe un parque vaciar su backlog hacia central tras reconectarse?
Respuesta del user: El tiempo que tarde — no hay restricción de ventana temporal.

## [arquitecto-scada-distribuido]
Pregunta: ¿Queréis priorización durante resincronización (alarmas primero, luego telemetría reciente, histórico al final)?
Respuesta del user: No — el histórico se va sincronizando y central va procesando la información conforme la recibe. Todo depende del histórico tal como llega.

## [ingeniero-datos-series-temporales]
Pregunta: ¿Orden de magnitud de señales activas por parque? ¿Cuántas van realmente a 1s de forma sostenida?
Respuesta del user: Dependiendo — es una caja negra, no lo sabemos aún.

## [ingeniero-datos-series-temporales]
Pregunta: ¿Preservar orden estricto por señal durante replay, o basta consistencia eventual con deduplicación?
Respuesta del user: Orden — entiende que una plataforma como RabbitMQ mantiene el orden.

## [ingeniero-datos-series-temporales]
Pregunta: ¿La TSDB local actúa también como buffer de reenvío, o separar persistencia local y cola de sincronización?
Respuesta del user: Separar — cola tipo RabbitMQ para la sincronización (consistente con la respuesta anterior).

# Lead notes — cliente-renovables/scada-general · 2026-05-26-arquitectura-concreta

## Directrices del user durante el debate

- `edge-reporter` existe como servicio propio desde fase 1.
- `asset-context-service` es servicio independiente desde fase 1.
- Exposición REST a terceros: no se sabe si incluirá suscripciones/event push — dejar abierto.
- Mecanismo concreto del puente parque→central: fijar uno (no dejar ambiguo).
- Edge opera autónomo incluso si falla control-plane-api central.
- Telemetría subida incluye TODO: muestras, estados Y alarmas locales como stream propio.
- Clave única de muestra: park_id + asset_id + signal_id + source_timestamp — ponerlo en el entregable.
- Prioridad de consulta del histórico central: no se sabe todavía → dejarlo abierto.
- Trazabilidad técnica (protocol, source_endpoint): misma BD pero schema relacional, no en hypertable.
- Incluir patrón outbox/inbox idempotente explícitamente.
- Distinguir deployables reales de capacidades internas en el entregable.
- RabbitMQ en edge: mínimo austero (no Shovel sofisticado).
- Incluir matriz de compatibilidad de versiones edge↔central.

## Intervenciones del lead (mediación)

- No hubo Ronda C' (shuttle diplomacy). Las críticas de B' convergieron en señalar los mismos gaps (dual-write, bridge ambiguo, naming). Las respuestas del user en escalaciones B' resolvieron todos los puntos divergentes. Se fue directo a posiciones finales (D').

## Conflictos abiertos

- Ninguno sustancial. Los 4 expertos convergen en la misma dirección tras las respuestas del user.

## Contexto fuera de archivos

- La prioridad de consulta del histórico central (último valor vs rangos vs agregados) queda abierta — el user dice "no se sabe". El moderador debe reflejar esto como decisión pendiente sin forzar una respuesta.

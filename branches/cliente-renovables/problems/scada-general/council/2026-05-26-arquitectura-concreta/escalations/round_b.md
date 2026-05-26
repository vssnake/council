# Escalaciones — Ronda B' · 2026-05-26-arquitectura-concreta

## Preguntas y respuestas

1. **¿Mecanismo concreto del puente parque→central?** → Sí, fijar uno concreto (no dejarlo ambiguo).
2. **¿Edge opera autónomo incluso si falla control-plane-api central?** → Sí, dejarlo explícito.
3. **¿Telemetría subida incluye alarmas locales?** → Todo: muestras, estados Y eventos de alarma local como stream propio.
4. **¿Fijar clave única de muestra (park_id + asset_id + signal_id + source_timestamp)?** → Sí, está bien ponerlo.
5. **¿Histórico central prioriza último valor, rangos crudos o agregados largos?** → No se sabe todavía.
6. **¿Trazabilidad técnica en historian principal o journal separado?** → En la misma base de datos pero relacional (no en la hypertable temporal).
7. **¿Incluir patrón outbox/inbox idempotente explícitamente?** → Sí, incluirlo.
8. **¿Distinguir deployables reales vs capacidades internas?** → Sí.
9. **¿RabbitMQ en edge mínimo austero o puerta a Shovel?** → Mínimo austero.
10. **¿Incluir matriz compatibilidad versiones edge↔central?** → Sí.

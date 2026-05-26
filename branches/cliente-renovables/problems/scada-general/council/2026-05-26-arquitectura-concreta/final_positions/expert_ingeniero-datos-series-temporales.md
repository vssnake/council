## Posición

Mi propuesta consolidada final para la capa temporal es la siguiente:

### 1) Deployables reales vs capacidades
**Deployables reales en planta**:
1. `scada-edge-runtime`
2. `timescaledb-edge`
3. `rabbitmq-edge`

**Deployables reales en central**:
1. `rabbitmq-central-ingress`
2. `central-telemetry-ingest`
3. `timescaledb-central`
4. `telemetry-query-api`
5. `asset-context-service`
6. `edge-reporter`

**Capacidades internas, no deployables aparte** dentro de `scada-edge-runtime`:
- normalización técnica
- escritura temporal local
- outbox publisher
- alarm/event exporter
- health/admin local

No recomiendo más despiece en edge.

### 2) Modelo de datos temporal
**Clave única de muestra**: `park_id + asset_id + signal_id + source_timestamp`.

**Hypertable raw principal** (`telemetry_samples`):
- `park_id`
- `asset_id`
- `signal_id`
- `source_timestamp`
- `ingest_timestamp`
- `value_num` / `value_text` / `value_bool`
- `quality_flag`
- `unit_code`
- `message_id`
- `schema_version`
- `is_late`

**Restricción única recomendada**:
- `UNIQUE (park_id, asset_id, signal_id, source_timestamp)`

**Proyecciones de lectura separadas**:
- `telemetry_latest_view`: último valor por señal
- `telemetry_agg_5m`, `telemetry_agg_1h`, `telemetry_agg_1d`: agregados para histórico largo

**Trazabilidad técnica**: misma base de datos, pero en **schema relacional separado**, no en la hypertable temporal. Propongo:
- `ops.ingestion_journal`
- `ops.duplicate_log`
- `ops.replay_batches`
- `ops.signal_trace`

Aquí sí pueden vivir `protocol`, `source_endpoint`, causa de degradación, intentos, errores y evidencias de replay.

### 3) Patrón de consistencia edge -> central
Debe quedar **explícito** el patrón **outbox/inbox idempotente**.

**En edge**:
1. `scada-edge-runtime` escribe la muestra en `timescaledb-edge.telemetry_samples`.
2. En la misma transacción lógica escribe un registro en `ops.outbox_events`.
3. Un `outbox publisher` publica desde `ops.outbox_events` a `rabbitmq-edge`.

**En central**:
1. `central-telemetry-ingest` consume desde `rabbitmq-central-ingress`.
2. Registra recepción idempotente en `ops.inbox_events`.
3. Persiste en `timescaledb-central.telemetry_samples` con `UPSERT`/insert idempotente.
4. Hace `ack` al broker **solo después** de persistencia correcta.

Esto cierra el dual-write. Sin este patrón, la propuesta queda incompleta.

### 4) Flujo temporal y replay
Flujo recomendado:

```text
campo -> scada-edge-runtime -> timescaledb-edge + outbox -> rabbitmq-edge
      -> AMQPS saliente iniciado desde planta
      -> rabbitmq-central-ingress -> central-telemetry-ingest
      -> timescaledb-central -> telemetry-query-api / reporting / alarmado central
```

Políticas obligatorias:
- **Replay solo vive en broker/outbox**; **nunca** se reconstruye consultando la TSDB.
- **RabbitMQ en edge mínimo austero**: durable, simple, sin sofisticación inicial innecesaria.
- El edge sigue capturando y persistiendo aunque falle el control plane central.
- Se suben como streams separados: **muestras**, **estados** y **eventos de alarma local**.
- `is_late=true` cuando central persiste backlog fuera de tiempo operativo.
- Orden garantizado por partición lógica de señal/activo; idempotencia garantizada por clave única de muestra.

### 5) Stack TSDB
Mi recomendación sigue siendo **TimescaleDB** en edge y central, con el mismo contrato temporal en ambos niveles.

Lo que sí fijaría ya en el entregable:
- raw append-only como base
- vistas/proyecciones separadas para `latest` y agregados
- compresión y retención por horizontes
- consultas operativas contra vistas y agregados, no contra raw indiscriminadamente

**Prioridad de consulta del histórico central**: sigue **abierta**. No sabemos aún si debe optimizarse primero para `latest`, rangos raw o agregados largos; por tanto, la estrategia fina de índices/proyecciones debe quedar como decisión posterior de sizing funcional.

### 6) Interfaces de persistencia y lectura
**Persistencia edge**
- `scada-edge-runtime -> timescaledb-edge`: SQL/Npgsql, inserción idempotente
- `scada-edge-runtime -> ops.outbox_events`: misma BD, schema relacional
- `outbox publisher -> rabbitmq-edge`: AMQP durable

**Persistencia central**
- `rabbitmq-central-ingress -> central-telemetry-ingest`: AMQP
- `central-telemetry-ingest -> ops.inbox_events`: SQL/Npgsql
- `central-telemetry-ingest -> timescaledb-central`: SQL/Npgsql con insert idempotente

**Lectura central**
- `telemetry-query-api/latest`
- `telemetry-query-api/raw-range`
- `telemetry-query-api/aggregated-range`

No mezclar en una sola interfaz los tres patrones de consulta.

### 7) Compatibilidad de versiones edge <-> central
Sí recomiendo incluir una matriz mínima de compatibilidad:
- `schema_version` en cada mensaje
- compatibilidad hacia atrás de al menos una versión N-1 en central
- un edge no debe publicar un envelope que central no pueda aceptar idempotentemente
- cambios de contrato requieren rollout coordinado y rollback soportado

## Razones

- El user confirmó **outbox/inbox idempotente explícito** y debe reflejarse como patrón arquitectónico, no como detalle implícito.
- El user confirmó la **clave única de muestra** y con ella se cierra la deduplicación determinista.
- El user dejó abierta la **prioridad de consulta del histórico central**; por eso no conviene sobrefijar índices finales todavía.
- El user pidió **trazabilidad técnica en la misma BD pero relacional**, así que la propongo fuera de la hypertable temporal.
- El user pidió distinguir **deployables** de **capacidades**; en edge eso evita el despiece accidental.
- El user pidió **RabbitMQ mínimo austero**; por eso no propongo complejidad extra como base.
- El replay desde consultas a TSDB es un antipatrón aquí: mezcla historian con mecanismo de resincronización y complica consistencia.

## Supuestos

- La identidad técnica de señal (`park_id`, `asset_id`, `signal_id`) puede estabilizarse desde fase 1.
- Toda la telemetría relevante debe llegar a central, aunque llegue tarde.
- El historian central y el edge comparten modelo lógico aunque no necesariamente el mismo tuning físico.
- `asset-context-service` aporta semántica extendida fuera del raw temporal; no debe inflar la hypertable principal.

## Preguntas al user

- Ninguna adicional en esta ronda; dejo abierta únicamente la prioridad futura de consulta del histórico central porque el user indicó que aún no se conoce.
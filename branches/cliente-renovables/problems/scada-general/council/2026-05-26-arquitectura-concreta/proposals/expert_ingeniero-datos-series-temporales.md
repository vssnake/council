## Posición

Propongo concretar la arquitectura temporal alrededor de un patrón simple y verificable: **en planta, 3 unidades desplegables** —`scada-edge-runtime`, `timescale-edge`, `edge-sync-broker`— y **en central, una cadena explícita de ingestión temporal** —`central-telemetry-ingest`, `timescale-central`, `telemetry-query-api`, `temporal-retention-jobs`—. Esto mantiene la compacidad acordada para edge y aterriza el detalle que pide el follow-up [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/outcome.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-arquitectura-concreta/follow_up.md].

### 1) Catálogo propuesto de servicios temporales en planta (edge)

| Servicio / contenedor | Responsabilidad temporal concreta | Stack propuesto | Interfaces principales |
|---|---|---|---|
| `scada-edge-runtime` | Normaliza la telemetría recibida del concentrador multi-protocolo, asigna `signal_id` canónico, conserva `source_timestamp` y `quality_flag`, escribe en TSDB local y publica eventos listos para sincronización | .NET LTS Worker Service + ASP.NET Core Minimal API para health/admin + cliente AMQP + driver PostgreSQL | Escribe en `timescale-edge`; publica en `edge-sync-broker`; expone health/admin local |
| `timescale-edge` | Persistencia temporal local operativa e histórica de planta; base para consulta local, alarmado local y recuperación tras caída WAN | PostgreSQL + TimescaleDB | Recibe escritura idempotente desde `scada-edge-runtime`; lectura local para reporter/alarmado |
| `edge-sync-broker` | Buffer persistente de sincronización planta→central; desacopla captura local de disponibilidad WAN; mantiene orden por partición lógica de señal | RabbitMQ durable | Recibe publicación desde `scada-edge-runtime`; entrega a `central-telemetry-ingest` cuando hay conectividad |

**Esquema TSDB propuesto en planta**
- `telemetry_raw`: hypertable de grano fino con clave lógica `park_id + asset_id + signal_id + source_timestamp`; campos mínimos: `value_*`, `quality_flag`, `ingest_timestamp`, `message_id`, `schema_version`.
- `telemetry_last_value`: vista/materialización operativa para “último valor por señal” y pantallas locales.
- `telemetry_gap_log`: registro técnico de huecos, duplicados rechazados y retrasos detectados durante replay.
- `retention_policy`: política declarativa por familia de señales para separar conservación nativa y agregada.

### 2) Catálogo propuesto de servicios temporales en central

| Servicio / contenedor | Responsabilidad temporal concreta | Stack propuesto | Interfaces principales |
|---|---|---|---|
| `central-telemetry-ingest` | Termina el canal planta→central, valida envelope, aplica deduplicación determinista, preserva orden por señal y escribe en TSDB central | .NET LTS Worker Service + cliente AMQP + driver PostgreSQL | Consume desde `edge-sync-broker`; escribe en `timescale-central`; publica estado de ingestión |
| `timescale-central` | Historiador corporativo único para todos los parques; soporta consulta temporal, reporting y base para analítica futura | PostgreSQL + TimescaleDB | Escritura desde `central-telemetry-ingest`; lectura desde `telemetry-query-api`, reporting y alarmado central |
| `telemetry-query-api` | API de lectura temporal para dashboards, reporter y terceros REST; separa consumo de la escritura | ASP.NET Core Web API + driver PostgreSQL | Consulta `timescale-central`; expone REST de series, último valor y agregados |
| `temporal-retention-jobs` | Gestiona continuous aggregates, compresión, retención multinivel y refresco de vistas históricas | .NET Worker Service + SQL Jobs/automatización propia sobre TimescaleDB | Opera sobre `timescale-central`; publica métricas técnicas |
| `signal-catalog-service` | Catálogo de señales/activos y semántica extendida fuera de la TSDB; evita inflar cardinalidad temporal con metadatos de negocio | ASP.NET Core Web API + base relacional corporativa | Lo consultan `central-telemetry-ingest` y `telemetry-query-api` |

### 3) Flujo de datos temporal end-to-end

```text
Equipo de campo
  -> concentrador de adquisición multi-protocolo
  -> scada-edge-runtime
      -> write telemetry_raw en timescale-edge
      -> publish telemetry-envelope en edge-sync-broker
  -> enlace saliente planta->central (push asíncrono)
  -> central-telemetry-ingest
      -> dedup + control de orden + late-data marking
      -> write telemetry_raw_central en timescale-central
      -> refresh de agregados / consultas
  -> telemetry-query-api / reporting / alarmado central / analítica futura
```

**Contrato de mensaje temporal propuesto**
- Identidad técnica: `park_id`, `asset_id`, `signal_id`.
- Tiempo: `source_timestamp` como tiempo canónico de proceso y `ingest_timestamp` como tiempo técnico de recepción.
- Calidad: `quality_flag` en primer nivel, porque el user pidió preservarlo en central [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_a.md].
- Idempotencia: `message_id` estable por muestra para soportar replay sin duplicados.
- Orden: particionado lógico por señal/activo para respetar el requisito de orden durante resincronización [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_b.md].

### 4) Comunicación edge ↔ central

- **Patrón primario**: `push` saliente desde planta a central; no hay dependencia de conexión iniciada desde central, coherente con la restricción de red y con la respuesta explícita del user [fuente: branches/cliente-renovables/problems/scada-general/problem.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_a.md].
- **Backbone de telemetría**: AMQP persistente sobre RabbitMQ; **REST no debe ser el canal primario de telemetría** [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/outcome.md].
- **Uso de REST/HTTPS**: health, administración, consulta puntual, exposición de lectura y servicios a terceros; no para vaciar backlog temporal.
- **Topología lógica**:
  - planta: `scada-edge-runtime -> edge-sync-broker -> uplink sender`
  - central: `ingress broker/consumer -> central-telemetry-ingest -> timescale-central`
- **Patrón de resincronización**: store-and-forward con backlog completo; sin priorización funcional entre histórico y reciente, porque el user indicó que central procesa “tal como llega” [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_b.md].
- **Marcado de tardanza**: la central debe informar explícitamente retraso de datos cuando procese backlog, porque el user lo pidió [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_b.md].

### 5) Stack tecnológico temporal completo

| Capa | Tecnología propuesta | Papel |
|---|---|---|
| Runtime de servicios | .NET LTS | Restricción explícita del proyecto para los módulos [fuente: branches/cliente-renovables/problems/scada-general/problem.md] |
| Persistencia temporal edge | PostgreSQL + TimescaleDB | Historización local y lectura operacional |
| Persistencia temporal central | PostgreSQL + TimescaleDB | Historización corporativa y base de consumo |
| Sincronización edge→central | RabbitMQ | Store-and-forward persistente separado de la TSDB [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_b.md] |
| API de lectura | ASP.NET Core Web API | Exposición REST para dashboards/reporting/terceros |
| Jobs de retención/agregación | Worker .NET + SQL sobre TimescaleDB | Continuous aggregates, compresión y purge controlado |
| Plataforma edge | Kubernetes pequeño en planta | Restricción del proyecto [fuente: branches/cliente-renovables/problems/scada-general/problem.md] |
| Plataforma central | OpenShift | Restricción del proyecto [fuente: branches/cliente-renovables/problems/scada-general/problem.md] |
| Despliegue | Azure DevOps pipelines | Restricción del proyecto [fuente: branches/cliente-renovables/problems/scada-general/problem.md] |

### 6) Interfaces entre servicios

| Productor -> Consumidor | Qué expone | Cómo lo consume el siguiente | Protocolo |
|---|---|---|---|
| `scada-edge-runtime` -> `timescale-edge` | Escritura de muestras normalizadas e idempotentes | Inserción/UPSERT por lote pequeño o streaming transaccional | PostgreSQL wire protocol |
| `scada-edge-runtime` -> `edge-sync-broker` | Envelope temporal listo para sincronización | Publicación durable con clave de partición por señal/activo | AMQP |
| `edge-sync-broker` -> `central-telemetry-ingest` | Cola persistente de replay/near-real-time | Consumo con ack tras persistencia correcta en central | AMQP |
| `central-telemetry-ingest` -> `timescale-central` | Escritura idempotente, enriquecida con metadata mínima de ingestión | Inserción/UPSERT y registro de duplicados/llegada tardía | PostgreSQL wire protocol |
| `telemetry-query-api` -> `timescale-central` | Consultas de último valor, rangos y agregados | SQL parametrizado y vistas/continuous aggregates | PostgreSQL wire protocol |
| `central-telemetry-ingest` -> `signal-catalog-service` | Resolución de catálogo técnico/semántico | Consulta de metadatos fuera de la TSDB | REST interno |

### 7) Política temporal y de retención propuesta

- **Planta**: conservar resolución nativa en `timescale-edge` durante una ventana dimensionada para soportar caídas WAN de **días** [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_a.md]. No usar downsampling como sustituto de la subida completa, porque el user pidió **toda la telemetría a central** [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_a.md].
- **Central / hot**: retener dato nativo para explotación operativa y re-procesado cercano a origen.
- **Central / warm-cold**: activar continuous aggregates por horizontes temporales; la agregación histórica está explícitamente prevista “con el tiempo (unos años)” [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_a.md].
- **Regla de diseño**: `raw != aggregate`; los agregados son una proyección de consumo y conservación, no sustituyen el raw mientras el horizonte operativo lo requiera.
- **Disciplina obligatoria**: controlar cardinalidad de tags y mantener jerarquía de activos fuera de la hypertable principal; si no, la TSDB se degrada antes que la infraestructura.

## Razones

1. **Sigue el consenso verificado del run padre**: planta compacta, RabbitMQ separado de la TSDB, TimescaleDB como preferente sujeta a PoC y canónico técnico en origen [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/outcome.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/final_positions/expert_ingeniero-datos-series-temporales.md].
2. **Aterriza exactamente el gap del follow-up**: nombres de servicios, responsabilidades, flujo end-to-end, comunicaciones, stack e interfaces [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-arquitectura-concreta/follow_up.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-arquitectura-concreta/deliverable.md].
3. **Evita dos anti-patrones**: usar la TSDB como cola de reenvío y meter semántica de negocio completa dentro del modelo temporal.
4. **Protege los dos fallos que el user considera peores**: pérdida de datos y alarmas inconsistentes [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_a.md].
5. **Mantiene abierta la única reserva legítima**: TimescaleDB queda como opción preferente, pero no debe cerrarse definitivamente sin validación operativa real en edge [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/final_positions/expert_ingeniero-datos-series-temporales.md].

## Supuestos

- El parque seguirá desplegando una topología de **3 máquinas por parque** y un Kubernetes pequeño, por lo que edge debe priorizar simplicidad operacional [fuente: branches/cliente-renovables/problems/scada-general/problem.md].
- El sistema agregará datos de aproximadamente **20** plantas solares y aproximadamente **5** parques eólicos, así que la central debe diseñarse como historiador corporativo multi-site desde el inicio [fuente: branches/cliente-renovables/problems/scada-general/problem.md].
- La volumetría exacta de señales activas y de series a **1 s** sigue sin conocerse; por eso no cierro aún tamaños de partición, compresión ni ventanas exactas de retención [fuente: branches/cliente-renovables/problems/scada-general/problem.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_b.md].
- La exposición IEC 61850 a terceros puede ir después y no debe contaminar el backbone temporal de esta fase [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_a.md].
- Si la PoC de TimescaleDB en edge no supera backlog, replay y operación sostenida, debe reabrirse la decisión solo para planta, manteniendo el contrato temporal y el patrón de sincronización.

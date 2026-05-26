## Posición

Mi propuesta consolidada final es una arquitectura SCADA multinivel con **edge autónomo**, **bridge norte fijo por AMQPS**, y una distinción explícita entre **deployables reales** y **capacidades internas**. No recomiendo dejar el tramo edge→central ambiguo: el mecanismo concreto debe ser **`northbound-relay` dentro de `scada-edge-runtime`, consumiendo de `rabbitmq-edge` y publicando por `AMQPS` a `rabbitmq-central-ingress`**. No usaría `Shovel` en fase 1 porque el user ha pedido un RabbitMQ edge mínimo austero.

## Catálogo de Servicios en Planta (Edge)

#### Deployables reales en edge

| Deployable real | Responsabilidad | Tecnología principal | Dependencias |
|---|---|---|---|
| `scada-edge-runtime` | Concentrador OT único: adquisición multi-protocolo, normalización técnica, evaluación de alarmas locales, outbox, relay norte, caché de configuración, API local de operación | .NET 8 LTS (`Worker Service` + `ASP.NET Core`) | `timescaledb-edge`, `rabbitmq-edge` |
| `timescaledb-edge` | Historian local operativo y persistencia de referencia para operación de planta | PostgreSQL 16 + TimescaleDB | PVC local |
| `rabbitmq-edge` | Broker local austero para desacoplo y store-and-forward | RabbitMQ 3.13 durable, topología mínima | PVC local |
| `edge-reporter` | Reporting local desde fase 1, separado del runtime como pidió el user | .NET 8 LTS (`ASP.NET Core` + jobs) | `timescaledb-edge` |

#### Capacidades internas dentro de `scada-edge-runtime` (no deployables separados)

- `field-driver-host`: drivers OPC UA, Modbus TCP/RTU, MQTT, OPC DA.
- `technical-canonicalizer`: construye el canónico técnico de planta.
- `local-alarm-engine`: evalúa alarmas locales y las publica como stream propio.
- `edge-outbox-writer`: escribe muestra + outbox en la misma transacción.
- `northbound-relay`: drena `rabbitmq-edge` y publica a central por `AMQPS`.
- `config-cache`: mantiene última configuración válida; si falla central, edge sigue operando.
- `edge-ops-api`: health, backlog, estado de drivers y diagnóstico local.

## Catálogo de Servicios en Central

| Deployable real | Responsabilidad | Tecnología principal | Dependencias |
|---|---|---|---|
| `rabbitmq-central-ingress` | Punto de entrada de telemetría y alarmas desde todos los parques | RabbitMQ 3.13 sobre OpenShift | PVC central |
| `central-ingestion-service` | Consume streams, aplica patrón inbox idempotente, deduplica y persiste | .NET 8 LTS `Worker Service` | `rabbitmq-central-ingress`, `timescaledb-central` |
| `timescaledb-central` | Historian corporativo único para todos los parques | PostgreSQL 16 + TimescaleDB | Storage persistente |
| `asset-context-service` | Catálogo técnico/corporativo independiente desde fase 1 | .NET 8 LTS `ASP.NET Core` + schema relacional `catalog` | `timescaledb-central` |
| `central-alarm-service` | Consolida alarmado agregado y correlación multi-parque | .NET 8 LTS `Worker Service` | `timescaledb-central`, `asset-context-service` |
| `central-reporter` | Reporting interno/externo en central | .NET 8 LTS | `timescaledb-central`, `asset-context-service` |
| `third-party-api` | Exposición REST a terceros y frontends | .NET 8 LTS `ASP.NET Core` | `timescaledb-central`, `asset-context-service` |
| `control-plane-api` | Configuración, manifiestos y directivas; **no es dependencia operativa del edge** | .NET 8 LTS `ASP.NET Core` | `asset-context-service` |

**Capacidad futura, no backbone de fase 1**
- `iec61850-adapter`: adaptador de publicación posterior; no condiciona la columna vertebral edge→central.

## Flujo de Datos End-to-End

1. `scada-edge-runtime` lee equipos de campo por OPC UA / Modbus / MQTT / OPC DA.
2. Normaliza a un `TelemetryEnvelope v1` con clave lógica única de muestra: **`park_id + asset_id + signal_id + source_timestamp`**.
3. En una sola transacción SQL escribe:
   - `telemetry.raw_samples` (hypertable)
   - `integration.outbox` (tabla relacional)
4. Un dispatcher interno publica el outbox en `rabbitmq-edge`.
5. `local-alarm-engine` genera `AlarmEvent v1`; esos eventos se guardan en `alarms.local_events` y también salen por un stream propio (`alarm.local.v1`).
6. `northbound-relay` consume de `rabbitmq-edge` y publica por `AMQPS` a `rabbitmq-central-ingress`.
7. `central-ingestion-service` consume, registra recepción en `integration.inbox`, hace `UPSERT` idempotente en `telemetry.raw_samples`, marca `late_arrival` cuando aplique y solo entonces hace `ack`.
8. `central-alarm-service`, `central-reporter` y `third-party-api` consumen desde la persistencia central.

```text
Equipos de campo
  -> scada-edge-runtime
     -> telemetry.raw_samples (edge)
     -> integration.outbox (edge)
     -> rabbitmq-edge
        => northbound-relay => AMQPS => rabbitmq-central-ingress
           -> central-ingestion-service
              -> integration.inbox (central)
              -> telemetry.raw_samples (central)
              -> alarms.local_events (central stream recibido)
                 -> central-alarm-service
                 -> central-reporter
                 -> third-party-api
```

## Comunicación Edge ↔ Central

#### Mecanismo concreto del puente

- **Bridge fijo**: `northbound-relay` en `scada-edge-runtime`.
- **Origen**: consume de `rabbitmq-edge`.
- **Destino**: publica en `rabbitmq-central-ingress`.
- **Protocolo**: `AMQPS` con mTLS sobre la VPN existente.
- **Sentido**: siempre iniciado desde planta hacia central.
- **REST**: reservado a `control-plane-api`, consultas puntuales y operación; no se usa como backbone de telemetría.

#### Contratos de stream

- `telemetry.sample.v1`
- `telemetry.state.v1`
- `alarm.local.v1`

#### Reglas operativas cerradas

- El edge **opera autónomo incluso si `control-plane-api` no está disponible**.
- La configuración activa en edge es la **última versión válida cacheada**.
- El backlog WAN vive en `rabbitmq-edge` + `integration.outbox`, no en lecturas desde la hypertable.
- `ack` en central solo tras persistencia correcta en inbox + TSDB.
- Mensajes inválidos pasan a `DLQ` por tipo de stream.

#### Matriz de compatibilidad de versiones edge ↔ central

| Edge envelope | Central soportado | Regla |
|---|---|---|
| `v1.x` | `v1.x` | Compatibilidad total |
| `v1.x` | `v1.y` (`y > x`) | Permitida si solo añade campos opcionales |
| `v2.x` | `v1.x` | No permitida |
| `v1.x` | `v2.x` | Permitida solo si central mantiene parser backward compatible |

**Regla de despliegue**: primero central, después edge. Nunca al revés en cambios de contrato.

## Stack Tecnológico Completo

| Capa / función | Tecnología |
|---|---|
| Runtime servicios | .NET 8 LTS |
| APIs HTTP | ASP.NET Core |
| Adquisición OT | OPC UA client, Modbus TCP/RTU master, MQTT client, OPC DA adapter |
| Broker edge | RabbitMQ 3.13 mínimo austero |
| Broker central | RabbitMQ 3.13 |
| Transporte edge→central | AMQPS + mTLS |
| Historian edge/central | PostgreSQL 16 + TimescaleDB |
| Acceso .NET a BD | Npgsql |
| Orquestación edge | Kubernetes pequeño por parque |
| Orquestación central | OpenShift |
| CI/CD | Azure DevOps Pipelines + Helm |
| Observabilidad | OpenTelemetry + Prometheus-compatible metrics + logs estructurados |

#### Esquemas de datos en la misma BD (pero separados)

- `telemetry.*` → hypertables de series temporales.
- `integration.*` → tablas relacionales para `outbox`, `inbox`, `dead_letter`, `replay_cursor`.
- `ops_trace.*` → trazabilidad técnica relacional (`protocol`, `source_endpoint`, causa de rechazo, auditoría de replay, estado de backlog).
- `catalog.*` → contexto técnico/corporativo.
- `alarms.*` → eventos y estado de alarmas.

**Posición explícita**: la trazabilidad técnica va en la **misma base de datos**, pero en **schema relacional**, no en hypertable temporal.

## Interfaces entre Servicios

| Origen | Destino | Qué expone / transmite | Protocolo |
|---|---|---|---|
| Equipos de campo | `scada-edge-runtime` | muestras, estados, eventos de dispositivo | OPC UA / Modbus / MQTT / OPC DA |
| `scada-edge-runtime` | `timescaledb-edge` | `raw_samples`, `local_events`, cursores locales | SQL / PostgreSQL |
| `scada-edge-runtime` | `rabbitmq-edge` | `telemetry.sample.v1`, `telemetry.state.v1`, `alarm.local.v1` | AMQP 0-9-1 |
| `northbound-relay` | `rabbitmq-central-ingress` | replay + near-real-time | AMQPS |
| `rabbitmq-central-ingress` | `central-ingestion-service` | entrega desacoplada por stream y parque | AMQP 0-9-1 |
| `central-ingestion-service` | `timescaledb-central` | inserción idempotente + inbox + marcas de late data | SQL / PostgreSQL |
| `central-ingestion-service` | `asset-context-service` | lookup por lote/caché, nunca por muestra en hot path | REST interno |
| `timescaledb-central` | `central-alarm-service` | muestras, estados y stream de alarmas locales recibido | SQL / PostgreSQL |
| `timescaledb-central` | `central-reporter` | histórico, latest, agregados | SQL / PostgreSQL |
| `third-party-api` | terceros | consulta REST de series, estados y alarmas | HTTPS REST |
| `scada-edge-runtime` | `control-plane-api` | pull de configuración y releases | HTTPS REST |

## Razones

- Resuelve el requisito más importante del dominio: **autonomía real del edge** sin dependencia del plano central.
- Distingue lo que el cliente necesita ver: **deployables reales** frente a capacidades internas.
- Cierra el dual-write con un patrón **outbox/inbox idempotente** explícito.
- Fija un bridge concreto y austero: relay de aplicación + RabbitMQ local mínimo, sin sofisticación prematura.
- Sube a central **muestras, estados y alarmas locales** como streams separados.
- Evita meter trazabilidad técnica en la hypertable, manteniendo la TSDB limpia y el troubleshooting viable.
- Mantiene `asset-context-service` independiente desde fase 1, pero fuera del hot path por muestra.

## Supuestos

- `edge-reporter` y `asset-context-service` deben existir desde fase 1, tal como confirmó el user.
- La VPN permite tráfico saliente `AMQPS` y `HTTPS` desde planta hacia central.
- La volumetría exacta sigue por validar; esta propuesta fija arquitectura, no sizing final.
- IEC 61850 queda como capacidad posterior, no como restricción de backbone.

## Preguntas al user

- Ninguna adicional en esta ronda.
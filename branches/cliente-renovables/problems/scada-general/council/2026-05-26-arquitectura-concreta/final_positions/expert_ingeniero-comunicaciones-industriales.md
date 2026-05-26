## Posición

Mi propuesta consolidada final para el entregable es la siguiente:

### 1) Comunicación edge ↔ central: mecanismo cerrado y concreto
El puente **parque → central** debe fijarse como un patrón **push saliente AMQPS** iniciado desde planta, sin dependencias de entrada desde central y **sin usar Shovel/Federation ni topologías sofisticadas de RabbitMQ**.

Propongo este recorrido operativo:

```text
Equipos OT -> scada-edge-runtime -> TimescaleDB Edge (raw + outbox relacional)
                                   -> rabbitmq-edge (cola durable local)
                                   -> uplink-drainer dentro de scada-edge-runtime
                                   => AMQPS saliente => rabbitmq-central-ingress
                                   -> telemetry-ingestion-service
                                   -> TimescaleDB Central (raw + inbox relacional + trazabilidad)
```

Regla explícita: **el edge sigue capturando, historizando, alarmando y acumulando backlog aunque falle por completo cualquier control-plane-api central**. El plano de control es auxiliar; el plano de datos debe operar autónomamente.

### 2) Deployables reales vs capacidades internas
Para evitar ambigüedad en cliente, distingo así:

#### Deployables reales en planta (fase 1)
| Deployable | Responsabilidad | Tecnología |
|---|---|---|
| `scada-edge-runtime` | adquisición multi-protocolo, normalización técnica, alarmado local, reporter local, gestión de outbox, drenaje WAN | .NET LTS |
| `timescaledb-edge` | historian local y persistencia técnica relacional | PostgreSQL + TimescaleDB |
| `rabbitmq-edge` | buffer durable local de sincronización | RabbitMQ austero |

#### Capacidades internas de `scada-edge-runtime` (no deployables separados)
- `field-driver-host`
- `technical-canonicalizer`
- `local-alarm-engine`
- `outbox-relay`
- `uplink-drainer`
- `edge-reporter`
- `edge-config-api`

Esto deja claro que **edge-reporter** y la lógica de uplink existen desde fase 1, pero **viven dentro del runtime**, no como microservicios independientes.

#### Deployables mínimos en central relevantes para esta capa
| Deployable | Responsabilidad | Tecnología |
|---|---|---|
| `rabbitmq-central-ingress` | terminador del canal AMQPS desde parques | RabbitMQ |
| `telemetry-ingestion-service` | consumo idempotente, validación de envelope, persistencia central | .NET LTS |
| `timescaledb-central` | historian corporativo + esquema relacional técnico | PostgreSQL + TimescaleDB |
| `asset-context-service` | catálogo técnico/corporativo de activos y señales | .NET LTS + BD relacional |
| `central-alarm-service` | vista agregada y correlación de alarmas en central | .NET LTS |

### 3) Stack de adquisición y contrato técnico de salida
El stack OT que sí pondría en el documento es:
- **Adquisición**: `OPC UA`, `Modbus TCP/RTU`, `MQTT` cuando el equipo ya publique, `OPC DA` solo detrás de adaptador legacy.
- **Runtime**: `.NET LTS Worker Services`.
- **Persistencia edge/central**: `PostgreSQL + TimescaleDB`.
- **Mensajería**: `RabbitMQ` con colas durables simples; sin features avanzadas como base de la propuesta.
- **APIs**: `ASP.NET Core` sobre HTTPS solo para operación, configuración y consumo, nunca como backbone de telemetría.

El canónico técnico que sale de planta debe incluir, como mínimo:
- `park_id`
- `asset_id`
- `signal_id`
- `value`
- `unit`
- `source_timestamp`
- `ingest_timestamp`
- `quality_flag`
- `schema_version`
- `sample_key`

La **clave única de muestra** queda fijada como:

`park_id + asset_id + signal_id + source_timestamp`

Eso debe viajar en el envelope y además gobernar la deduplicación central.

### 4) Telemetría que sube a central
Sube **todo** lo que el user confirmó:
1. **muestras**;
2. **estados**;
3. **eventos de alarma local como stream propio**.

No recomiendo mezclar semánticamente alarmas dentro del stream de muestras. Propongo dos streams lógicos bajo el mismo backbone:
- `telemetry.samples`
- `telemetry.local-alarms`

Ambos viajan por el mismo puente AMQPS y ambos se persisten en central con disciplina idempotente.

### 5) Outbox / inbox idempotente
Esto debe quedar escrito de forma explícita en el entregable:

#### En edge
La escritura de una muestra y el alta de su evento pendiente deben hacerse en la **misma base de datos** y en la **misma transacción local**, usando un **schema relacional** dentro de la misma instancia PostgreSQL/TimescaleDB:
- hypertable temporal para `telemetry_raw`
- tablas relacionales para `outbox_messages`, `uplink_attempts`, `protocol_trace`

El `outbox-relay` publica desde `outbox_messages` hacia `rabbitmq-edge`.

#### En central
`telemetry-ingestion-service` consume desde `rabbitmq-central-ingress` y persiste en una única transacción:
- inserción/upsert del dato temporal;
- registro en `inbox_messages`;
- trazabilidad técnica en tablas relacionales.

Regla de oro: **ack solo después de persistencia correcta**.

Con esto evitamos el dual-write implícito y cerramos la idempotencia extremo a extremo.

### 6) Trazabilidad técnica: misma BD, schema relacional
Confirmo la decisión del user: la trazabilidad técnica debe vivir en la **misma base de datos** que el historian, pero en **schema relacional separado**, no en la hypertable temporal principal.

Propongo separar así:
- `ts.telemetry_raw`
- `ts.telemetry_local_alarms`
- `rel.outbox_messages`
- `rel.inbox_messages`
- `rel.protocol_trace`
- `rel.delivery_attempts`
- `rel.version_compatibility`

`protocol`, `source_endpoint`, causa de degradación, intentos de entrega, duplicados y errores de parsing deben ir a `rel.*`, no como tags/campos de alta cardinalidad en la hypertable raw.

### 7) Interfaces concretas entre servicios
| Origen | Destino | Qué pasa | Protocolo |
|---|---|---|---|
| Equipos OT | `scada-edge-runtime` | lecturas, estados, eventos de origen | OPC UA / Modbus / MQTT / OPC DA |
| `scada-edge-runtime` | `timescaledb-edge` | raw telemetry, alarm events, outbox | PostgreSQL |
| `outbox-relay` | `rabbitmq-edge` | publicación durable local | AMQP 0-9-1 |
| `uplink-drainer` | `rabbitmq-central-ingress` | envío parque→central | AMQPS saliente |
| `rabbitmq-central-ingress` | `telemetry-ingestion-service` | entrega desacoplada por stream | AMQP 0-9-1 |
| `telemetry-ingestion-service` | `timescaledb-central` | persistencia temporal + inbox + trazabilidad | PostgreSQL |
| `telemetry-ingestion-service` | `asset-context-service` | enriquecimiento no bloqueante, preferiblemente cacheado | REST interno / lectura cacheada |
| `central-alarm-service` | `timescaledb-central` | lectura de muestras y eventos de alarma local | PostgreSQL |

Regla importante: **la aceptación inicial de telemetría no debe bloquearse por una llamada síncrona al catálogo**. Si hace falta contexto, se resuelve con caché o enriquecimiento posterior.

### 8) Compatibilidad de versiones edge ↔ central
También dejaría una regla de propuesta, no de implementación detallada:
- el envelope lleva `schema_version`;
- central debe soportar al menos `N` y `N-1`;
- un edge desalineado nunca debe perder captura local por incompatibilidad con central; como mucho acumula backlog hasta upgrade o compatibilidad restaurada.

## Razones

- Cierra el hueco principal del follow-up: **cómo es exactamente el puente parque→central**.
- Respeta la restricción OT real: la central no entra en planta; el tráfico operativo se inicia desde edge.
- Hace explícito que **edge opera autónomo** aunque falle el control plane central.
- Evita el antipatrón de usar la TSDB como mecanismo de replay WAN.
- Evita también otro antipatrón: meter `protocol` y trazas de transporte en la hypertable temporal.
- Mantiene **RabbitMQ mínimo austero**, suficiente para store-and-forward, sin vender complejidad innecesaria.
- Distingue por fin **deployables** de **capacidades internas**, que es justo donde varias propuestas estaban mezclando niveles.
- Deja una propuesta defendible ante cliente: concreta en servicios, interfaces, protocolo y disciplina de dato.

## Supuestos

- El enlace VPN permitirá exponer en central un endpoint AMQPS y endpoints HTTPS de operación.
- La volumetría exacta por señal aún no está cerrada; por eso no fijo tamaños ni retenciones numéricas.
- `OPC DA` solo se mantendrá donde exista legado real; no debe condicionar la arquitectura global.
- IEC 61850 queda como adaptador de publicación en central, fuera del backbone inicial.
- La operación en planta prioriza simplicidad: si una decisión aumenta complejidad sin mejorar claramente resiliencia, debe descartarse.

## Preguntas al user

- Ninguna adicional en esta ronda.
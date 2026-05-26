## Posición
Aterrizo el follow-up en una propuesta concreta de comunicaciones OT: en planta recomiendo mantener el shape compacto ya validado en el run padre —**máximo 3 unidades desplegables**: `scada-edge-runtime`, `timescaledb-edge` y `rabbitmq-edge` [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/outcome.md].

Dentro de ese shape, propongo nombrar y fijar estos servicios e interfaces de mi competencia:

### Catálogo concreto en planta (ámbito comunicaciones/adquisición)
| Servicio / contenedor | Responsabilidad principal | Tecnología concreta propuesta | Interfaces principales |
|---|---|---|---|
| `scada-edge-runtime` | Concentrador único OT: adquisición multi-protocolo, normalización técnica, evaluación de calidad, publicación local y envío norte | .NET LTS + Worker Services; drivers `OPC UA`, `Modbus TCP/RTU`, `OPC DA` mediante adaptador Windows si aplica, `MQTT` para equipos que ya publiquen | Sur: clientes de protocolo industrial hacia equipos; Este: PostgreSQL/Npgsql hacia TimescaleDB; Norte local: AMQP 0-9-1 hacia RabbitMQ; Operación: HTTPS REST solo para configuración/health |
| `timescaledb-edge` | Historian local operativo y buffer de consulta local; no hace de cola de sincronización | TimescaleDB sobre PostgreSQL | Escritura desde `scada-edge-runtime`; lectura local para alarmado, reporter local y soporte operativo |
| `rabbitmq-edge` | Cola persistente de resincronización y desacoplamiento WAN; retiene backlog cuando no hay enlace | RabbitMQ con colas/quorum queues persistentes si la PoC de edge lo soporta; si no, colas durables clásicas [verificar en PoC] | Recibe AMQP desde `scada-edge-runtime`; reenvía a central mediante enlace saliente iniciado desde planta |

### Submódulos internos que conviene nombrar dentro de `scada-edge-runtime`
- `field-driver-host`: agenda de polling/suscripción por protocolo, respetando la restricción de mono-conexión en ciertos equipos [fuente: branches/cliente-renovables/problems/scada-general/problem.md].
- `technical-canonicalizer`: convierte cada lectura a un canónico técnico mínimo con `site_id`, `asset_id`, `signal_id`, unidad normalizada, `source_timestamp`, `ingest_timestamp`, `quality_flag`, `protocol`, `source_endpoint` [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/outcome.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_a.md].
- `historian-writer`: persiste en `timescaledb-edge`.
- `northbound-publisher`: publica el evento canónico en `rabbitmq-edge` con clave de partición estable por señal para preservar orden por señal en replay [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_b.md].
- `edge-ops-api`: expone configuración, health, diagnóstico de drivers y backlog; no es canal primario de telemetría [fuente: branches/cliente-renovables/problems/scada-general/problem.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/outcome.md].

### Catálogo concreto en central (solo interfaz con adquisición/ingesta)
| Servicio / contenedor | Responsabilidad principal | Tecnología concreta propuesta | Interfaces principales |
|---|---|---|---|
| `rabbitmq-central-ingress` | Punto de entrada de mensajes desde parques; desacopla recepción WAN de consumidores corporativos | RabbitMQ en OpenShift | Recibe enlace saliente desde `rabbitmq-edge`; entrega AMQP a consumidores centrales |
| `telemetry-ingestion-service` | Valida envelope, deduplica, marca retraso y enruta a persistencia central | .NET LTS + Worker Service + cliente RabbitMQ | Consume AMQP desde `rabbitmq-central-ingress`; escribe a TimescaleDB central; publica eventos de retraso/backlog |
| `telemetry-query-api` | API de consulta para capas superiores; no ingiere desde planta | ASP.NET Core Web API | Consulta TimescaleDB central vía SQL/Npgsql; expone REST/JSON a dashboards, reporting y terceros |
| `third-party-export-adapter` | Adaptador de exposición hacia REST externo o IEC 61850 en fase posterior; no conecta a planta | .NET LTS; IEC 61850 como adaptador específico posterior | Consume del modelo central, nunca del edge directamente [fuente: branches/cliente-renovables/problems/scada-general/problem.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_a.md] |

## Razones
### Comunicación edge ↔ central que sí recomendaría escribir en el entregable
Propongo fijar esta topología lógica:

```text
Equipos/PLCs/RTUs
  -> (OPC UA / Modbus / OPC DA / MQTT)
scada-edge-runtime
  -> TimescaleDB Edge
  -> RabbitMQ Edge
      => enlace saliente AMQPS iniciado desde planta =>
         RabbitMQ Central Ingress
           -> telemetry-ingestion-service
             -> TimescaleDB Central
               -> APIs / dashboards / reporting / terceros
```

Mi recomendación concreta es que el tramo **parque -> central** se describa como **AMQPS saliente iniciado desde planta sobre la VPN existente**, con `RabbitMQ Shovel` o patrón equivalente de puente saliente gestionado desde el lado edge. Eso aterriza dos restricciones verificadas: la transmisión la inicia planta y la WAN puede cortarse durante días [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_a.md; fuente: branches/cliente-renovables/problems/scada-general/problem.md].

No recomiendo que el documento deje esto en “RabbitMQ” genérico. Debe decir explícitamente:
- **la cola persistente vive también en planta**, separada de la TSDB [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_b.md];
- **el enlace WAN lo drena un publicador saliente desde parque**, no una conexión iniciada desde central [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_a.md];
- **REST queda relegado** a configuración, health-check, consulta puntual y APIs de consumo, no al backbone de telemetría [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/outcome.md].

### Interfaces entre servicios que conviene dejar escritas
Propongo este inventario mínimo de interfaces:

| Origen | Destino | Qué intercambian | Protocolo / estilo |
|---|---|---|---|
| Equipos de campo | `scada-edge-runtime` | Lecturas, estados, eventos según capacidad del equipo | OPC UA client/server, Modbus TCP/RTU master, OPC DA client, MQTT subscriber |
| `scada-edge-runtime` | `timescaledb-edge` | Muestras normalizadas y eventos de calidad | PostgreSQL wire protocol vía Npgsql |
| `scada-edge-runtime` | `rabbitmq-edge` | Evento canónico de telemetría listo para replay | AMQP 0-9-1 persistente |
| `rabbitmq-edge` | `rabbitmq-central-ingress` | Replicación/store-and-forward de backlog y datos en tiempo casi real | AMQPS saliente iniciado desde planta |
| `rabbitmq-central-ingress` | `telemetry-ingestion-service` | Entrega desacoplada de mensajes por parque/señal | AMQP 0-9-1 |
| `telemetry-ingestion-service` | TSDB central | Inserción idempotente + marcas de retraso | PostgreSQL wire protocol vía Npgsql |
| `telemetry-query-api` | consumidores centrales/terceros | Consulta de series, estados, alarmas y metadatos | HTTPS REST/JSON |
| `third-party-export-adapter` | terceros | Exposición desacoplada del modelo central | REST y, cuando aplique, IEC 61850 posterior [fuente: branches/cliente-renovables/problems/scada-general/problem.md] |

### Qué canónico técnico debe salir de planta
En mi ámbito, el documento debe fijar que **la normalización en planta no es semántica de negocio completa**, pero sí un canónico técnico obligatorio. Propongo este mínimo:
- identificador estable de parque, activo y señal;
- valor tipado;
- unidad ya convertida a convención común cuando aplique;
- `source_timestamp` y `ingest_timestamp` separados;
- `quality_flag` y causa de degradación cuando exista;
- protocolo y endpoint de origen para trazabilidad técnica [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/outcome.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/final_positions/expert_ingeniero-comunicaciones-industriales.md].

Esto evita el antipatrón de “normalización tardía” y permite que central reciba toda la telemetría sin perder contexto temporal ni calidad [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_a.md].

### Stack tecnológico que sí pondría negro sobre blanco
Para no dejar la parte OT demasiado abstracta, yo escribiría así el stack de esta capa:
- **Runtime de servicios**: .NET LTS en contenedores Linux para el concentrador principal [fuente: branches/cliente-renovables/problems/scada-general/problem.md].
- **Drivers OT preferentes**: OPC UA y Modbus como caminos principales; OPC DA encapsulado detrás de adaptador si sigue habiendo legacy; MQTT solo cuando el equipo ya publique por ese canal, no como sustituto de adquisición industrial [fuente: branches/cliente-renovables/problems/scada-general/problem.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-arquitectura-concreta/experts/ingeniero-comunicaciones-industriales/persona.md].
- **Persistencia edge y central**: TimescaleDB, coherente con la convergencia del run padre y con la preferencia de una sola TSDB [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/outcome.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_a.md].
- **Mensajería de sincronización**: RabbitMQ, separado de TSDB y del API plane [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/user_directives.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_b.md].
- **Plano de consulta/exposición**: ASP.NET Core Web API sobre HTTPS para servicios de consumo y operación.
- **Interfaz futura a terceros**: adaptador IEC 61850 solo en central y como fase posterior [fuente: branches/cliente-renovables/problems/scada-general/problem.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_a.md].

### Qué desafiaría del outcome padre si queda ambiguo
Si el outcome refinado dejase “edge ↔ central” como una simple flecha RabbitMQ, yo lo desafiaría: falta decir **quién inicia la conexión**, **dónde vive el backlog**, **qué contrato mínimo lleva cada mensaje** y **qué interfaces quedan reservadas a REST**. Sin esas cuatro concreciones, el cliente sigue viendo una arquitectura correcta en abstracto pero no presentable como propuesta técnica.

## Supuestos
- Asumo que el enlace parque -> central seguirá siendo únicamente saliente para telemetría, tal como respondió el user [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_a.md].
- Asumo que toda la telemetría debe llegar a central y que el replay debe preservar orden por señal [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_a.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_b.md].
- Asumo que la separación TSDB local / cola de sincronización queda cerrada y no se va a intentar usar TimescaleDB como mecanismo de replay WAN [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_b.md].
- est. [verificar antes de decidir]: que `RabbitMQ Shovel` o mecanismo equivalente tiene encaje operativo suficiente en el pequeño Kubernetes de planta; si no, mantener RabbitMQ local pero mover la lógica de drenaje al `northbound-publisher` del runtime.
- est. [verificar antes de decidir]: que el porcentaje de equipos legacy con dependencia real de OPC DA no obliga a aislar un adaptador Windows separado del runtime principal.

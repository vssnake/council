## Posición
Propongo aterrizar la arquitectura en una forma **compacta en planta** y **modular en central**. En planta, cada parque despliega tres unidades operativas: **Edge Connector**, **Edge Historian** y **Edge Message Gateway**. En central, la plataforma se organiza en servicios separados para ingestión, historización, alarmado agregado, reporting y exposición a terceros. El enlace planta → central debe ser **push, asíncrono y store-and-forward**, con **RabbitMQ** como canal de sincronización y **TimescaleDB** como TSDB preferente en ambos niveles, tal como quedó validado en el run padre [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/outcome.md].

El valor de esta propuesta es que ya no habla de “módulos” de forma abstracta, sino de **servicios nombrados, responsabilidades concretas, interfaces explícitas y stack propuesto** para presentar al comité técnico del cliente [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-arquitectura-concreta/follow_up.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-arquitectura-concreta/deliverable.md].

## Razones
- El run padre ya cerró que en planta no convienen microservicios finos y que el shape objetivo es un máximo de 3 unidades desplegables [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/outcome.md].
- El user confirmó que la planta debe seguir operando durante días sin WAN, que toda la telemetría debe llegar a central, y que la sincronización debe separar TSDB local y cola de reenvío [fuente: branches/cliente-renovables/problems/scada-general/problem.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_a.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_b.md].
- El follow-up pide precisamente concretar nombres de servicios, flujo end-to-end, topología edge↔central, stack e interfaces; por tanto, la refinación correcta no es reabrir la decisión macro, sino **bajarla a arquitectura propuesta para cliente** [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-arquitectura-concreta/follow_up.md].

## Catálogo de Servicios en Planta (Edge)
| Servicio / contenedor | Responsabilidad principal | Tecnología principal propuesta | Dependencias |
|---|---|---|---|
| **edge-connector** | Concentrador único de adquisición multi-protocolo. Lee equipos de campo, unifica polling/suscripción, normaliza al canónico técnico y añade `source_timestamp`, `quality`, `site_id`, `asset_id`, `signal_id`. | .NET LTS Worker Service + drivers/protocol adapters (`OPC UA .NET Standard`, `MQTTnet`, librería Modbus para .NET; adaptador OPC DA encapsulado si aplica) [fuente: branches/cliente-renovables/problems/scada-general/problem.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/outcome.md] | Red industrial local; catálogo local de señales; `edge-historian`; `edge-alarm-engine`; `edge-message-gateway` |
| **edge-historian** | Persistencia local de series temporales y consulta operativa en planta. Guarda dato crudo y agregados operativos locales; sirve de base para replay si hay validaciones o relecturas locales. | TimescaleDB como TSDB preferente, sujeto a PoC operativa en edge [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/outcome.md] | Volumen persistente local; `edge-connector`; `edge-alarm-engine`; `edge-reporter` |
| **edge-message-gateway** | Store-and-forward hacia central. Publica telemetría y eventos de forma persistente, ordenada por señal y tolerante a cortes WAN. Gestiona reintentos, backpressure y confirmación de entrega. | RabbitMQ en modo durable + publicadores/consumidores .NET en `edge-connector` y `edge-sync-worker` [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/outcome.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/user_directives.md] | WAN/VPN saliente; `central-ingestion-gateway` |
| **edge-alarm-engine** | Evalúa reglas locales sobre el dato disponible en planta y genera alarmas locales sin depender de central. Mantiene estado local de alarma y journal operativo. | Módulo .NET LTS embebido en `edge-connector` o sidecar lógico dentro del mismo despliegue compacto [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/final_positions/expert_arquitecto-scada-distribuido.md] | `edge-historian`; catálogo de reglas; HMI local |
| **edge-reporter** | Genera extractos e informes operativos locales cuando aplique y prepara exportaciones puntuales. | ASP.NET Core minimal API + motor de plantillas/reporting .NET [est.] [verificar antes de decidir] | `edge-historian`; `edge-alarm-engine` |
| **edge-admin-api** | Configuración local de conectores, health, inventario lógico de señales, diagnóstico y operaciones de soporte. No transporta telemetría a central. | ASP.NET Core Web API dentro del mismo despliegue de aplicación consolidada [fuente: branches/cliente-renovables/problems/scada-general/problem.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/outcome.md] | `edge-connector`; `edge-historian`; autenticación máquina/operador |

**Lectura arquitectónica de planta**: aunque la tabla enumera capacidades lógicas, operativamente recomiendo empaquetarlas en **tres unidades desplegables**: (1) `edge-runtime` = `edge-connector` + `edge-alarm-engine` + `edge-reporter` + `edge-admin-api`; (2) `edge-historian`; (3) `edge-message-gateway` [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/outcome.md].

## Catálogo de Servicios en Central
| Servicio / contenedor | Responsabilidad principal | Tecnología principal propuesta | Dependencias |
|---|---|---|---|
| **central-ingestion-gateway** | Punto de entrada desde parques. Termina conexiones salientes desde edge, valida identidad del parque, recibe mensajes y los enruta al bus corporativo. | RabbitMQ clusterizado en OpenShift + adaptador .NET LTS de entrada [fuente: branches/cliente-renovables/problems/scada-general/problem.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/outcome.md] | VPN/WAN; `canonical-ingestor`; observabilidad |
| **canonical-ingestor** | Valida el canónico técnico, deduplica, preserva orden por señal, etiqueta dato tardío y publica a persistencia/servicios consumidores. | .NET LTS Worker Service + librería AMQP + validación de esquema interna [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/outcome.md] | `central-ingestion-gateway`; `central-historian`; `central-alarm-service` |
| **central-historian** | Historización corporativa de toda la telemetría recibida, consultas temporales, agregados de medio/largo plazo y base para reporting y analítica. | TimescaleDB sobre OpenShift, misma tecnología que en planta si la PoC edge se confirma [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/outcome.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_a.md] | Almacenamiento persistente; `canonical-ingestor` |
| **central-alarm-service** | Recalcula/deriva alarmas agregadas de central con la misma lógica funcional basada en el dato disponible en central; informa retrasos de dato cuando corresponda. | .NET LTS Worker Service + ASP.NET Core API [fuente: branches/cliente-renovables/problems/scada-general/problem.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_a.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_b.md] | `central-historian`; `canonical-ingestor`; notificación/UI |
| **reporting-service** | Generación de informes internos/externos y datasets de consumo para cliente y terceros. | ASP.NET Core + jobs .NET programados [est.] [verificar antes de decidir] | `central-historian`; `central-alarm-service`; `asset-context-service` |
| **asset-context-service** | Mantiene jerarquía parque → activo → señal, metadatos técnicos, mapeos y parametrización que enriquecen el canónico técnico sin contaminar la ingesta. | ASP.NET Core + PostgreSQL/TimescaleDB schema relacional compartido [est.] [verificar antes de decidir] | `central-historian`; `third-party-api`; `reporting-service` |
| **third-party-api** | Exposición REST para terceros y consumidores corporativos. No ingiere desde planta; consume datos ya consolidados en central. | ASP.NET Core Web API + gateway/API management del cliente [fuente: branches/cliente-renovables/problems/scada-general/problem.md] | `central-historian`; `asset-context-service`; `central-alarm-service` |
| **operations-console** | Consola de operación: estado de parques, backlog de sincronización, salud de conectores, dato tardío, fallos de ingesta y auditoría. | Front-end web del cliente + APIs ASP.NET Core [est.] [verificar antes de decidir] | `central-ingestion-gateway`; `canonical-ingestor`; observabilidad |
| **ml-raw-export** | Publica datasets crudos y curados para futura analítica/ML sin acoplar esa fase a la ingesta operativa inicial. | Jobs .NET + exportación a almacenamiento corporativo [fuente: branches/cliente-renovables/problems/scada-general/problem.md] | `central-historian`; almacenamiento analítico corporativo |
| **iec61850-publisher** | Capa posterior de publicación IEC 61850 para terceros, separada del núcleo operativo inicial. | Adaptador específico a definir en fase posterior [fuente: branches/cliente-renovables/problems/scada-general/problem.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_a.md] | `central-historian`; `asset-context-service`; red/segmentación del cliente |

## Flujo de Datos End-to-End
**Narrativa propuesta**
1. Un equipo de campo publica o expone una variable por Modbus, OPC UA, MQTT u otro protocolo soportado [fuente: branches/cliente-renovables/problems/scada-general/problem.md].
2. `edge-connector` recoge el dato, lo convierte al canónico técnico mínimo y le añade identidad estable, `source_timestamp` y `quality` [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/outcome.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_a.md].
3. El dato se escribe en `edge-historian` para historización y consulta local; en paralelo, `edge-alarm-engine` evalúa reglas locales y genera eventos de alarma si aplica [fuente: branches/cliente-renovables/problems/scada-general/problem.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_a.md].
4. El mismo dato se publica al `edge-message-gateway`, que lo deja en cola durable para envío norte. Si la WAN cae, el dato permanece en cola hasta reconexión [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/outcome.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_b.md].
5. Cuando hay conectividad, el parque inicia la comunicación saliente y entrega mensajes al `central-ingestion-gateway`; la central no necesita abrir sesión hacia planta para recibir telemetría [fuente: branches/cliente-renovables/problems/scada-general/problem.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_a.md].
6. `canonical-ingestor` valida, deduplica, preserva orden por señal y marca telemetría tardía para que dashboards/reportes reflejen retraso de dato [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_b.md].
7. `central-historian` persiste toda la telemetría recibida; `central-alarm-service`, `reporting-service`, `third-party-api` y `ml-raw-export` consumen desde ahí o desde eventos derivados [fuente: branches/cliente-renovables/problems/scada-general/problem.md].

```text
Campo/PLC/RTU
   │  (Modbus / OPC UA / MQTT / OPC DA)
   ▼
edge-connector
   ├──> edge-historian (persistencia local)
   ├──> edge-alarm-engine (alarmado local)
   └──> edge-message-gateway (cola durable store-and-forward)
              │
              │  push saliente por VPN/WAN
              ▼
central-ingestion-gateway
   ▼
canonical-ingestor
   ├──> central-historian
   ├──> central-alarm-service
   ├──> reporting-service
   ├──> third-party-api
   └──> ml-raw-export
```

**Flujos de fallo relevantes**
- Si cae la WAN, la planta sigue capturando, almacenando y alarmando localmente; solo se difiere la sincronización a central [fuente: branches/cliente-renovables/problems/scada-general/problem.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/outcome.md].
- Si central recibe backlog atrasado, lo procesa conforme llega, preservando orden por señal e informando retraso, sin priorización especial de alarmas frente a histórico porque el user la descartó [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_b.md].

## Comunicación Edge ↔ Central
### Topología lógica
- **Topología recomendada**: estrella lógica con cada parque actuando como emisor autónomo y la central como concentrador de ingestión [fuente: branches/cliente-renovables/problems/scada-general/problem.md].
- **Dirección de sesión operativa**: parque → central para la transmisión de datos; no se debe requerir polling desde central para el flujo normal de telemetría [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_a.md].
- **Patrón de transporte**: `push + store-and-forward + confirmación de entrega + replay ordenado por señal` [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/outcome.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_b.md].

### Protocolo entre niveles
- **Backbone de telemetría**: AMQP sobre RabbitMQ entre `edge-message-gateway` y `central-ingestion-gateway` [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/outcome.md].
- **Uso de REST/HTTPS**: solo para `edge-admin-api`, operaciones de soporte, consulta puntual, reporting bajo demanda o APIs a terceros; no como canal principal de ingesta [fuente: branches/cliente-renovables/problems/scada-general/problem.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/outcome.md].
- **Contrato de mensaje**: sobre de telemetría con identificador estable de señal, valor, unidad/código de ingeniería, `source_timestamp`, `quality`, secuencia lógica por señal y metadatos mínimos de parque/activo [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/outcome.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_a.md].

### Endpoints lógicos propuestos
- `telemetry.raw` → cola/routing key para series temporales crudas.
- `telemetry.events` → cola/routing key para cambios de estado y eventos técnicos.
- `alarms.derived` → cola/routing key para eventos de alarma derivados en planta [est.] [verificar antes de decidir].
- `/api/edge/config` → configuración y diagnóstico local de `edge-admin-api` [est.] [verificar antes de decidir].
- `/api/v1/telemetry`, `/api/v1/alarms`, `/api/v1/reports` → APIs de consumo ya en central para terceros o front-ends [est.] [verificar antes de decidir].

## Stack Tecnológico Completo
| Capa / función | Tecnología propuesta | Papel en la solución |
|---|---|---|
| Runtime de servicios | .NET LTS | Base común obligatoria para módulos de planta y central [fuente: branches/cliente-renovables/problems/scada-general/problem.md] |
| APIs y servicios HTTP | ASP.NET Core | APIs de administración, reporting y exposición REST [fuente: branches/cliente-renovables/problems/scada-general/problem.md] |
| Procesos de background | .NET Worker Services | Conectores, ingesta, replay, alarmado y jobs |
| Adquisición OPC UA | OPC UA .NET Standard | Cliente industrial para lectura/suscripción [est.] [verificar antes de decidir] |
| Adquisición MQTT | MQTTnet | Cliente MQTT para dispositivos/event brokers [est.] [verificar antes de decidir] |
| Adquisición Modbus | Librería Modbus para .NET (p. ej. NModbus) | Driver de lectura Modbus TCP/RTU [est.] [verificar antes de decidir] |
| OPC DA legacy | Adaptador encapsulado específico para Windows o gateway de protocolo | Cobertura a legacy sin contaminar el resto del runtime [fuente: branches/cliente-renovables/problems/scada-general/problem.md] |
| Persistencia temporal | TimescaleDB | TSDB preferente en edge y central, pendiente de PoC edge [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/outcome.md] |
| Mensajería edge↔central | RabbitMQ | Store-and-forward, orden por señal y desacoplamiento temporal [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/outcome.md] |
| Acceso PostgreSQL | Npgsql + SQL nativo/Dapper o EF Core acotado | Acceso a TimescaleDB y metadatos [est.] [verificar antes de decidir] |
| Observabilidad | OpenTelemetry + Prometheus + Grafana | Trazas, métricas, backlog, latencia, health y capacidad [est.] [verificar antes de decidir] |
| Logs | OpenTelemetry logs / stack corporativo del cliente | Diagnóstico distribuido [est.] [verificar antes de decidir] |
| Contenerización | Docker/OCI images | Empaquetado de despliegue |
| Orquestación edge | Kubernetes ligero del parque | Ejecución en las 3 máquinas de planta [fuente: branches/cliente-renovables/problems/scada-general/problem.md] |
| Orquestación central | OpenShift | Plataforma central corporativa [fuente: branches/cliente-renovables/problems/scada-general/problem.md] |
| CI/CD | Azure DevOps Pipelines | Build, promoción y rollback coordinado [fuente: branches/cliente-renovables/problems/scada-general/problem.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_b.md] |
| Seguridad de transporte | TLS/mTLS máquina-a-máquina | Identidad y protección del enlace norte [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/outcome.md] |

## Interfaces entre Servicios
| Emisor | Receptor | Qué expone / transfiere | Cómo lo consume | Protocolo / patrón |
|---|---|---|---|---|
| Equipo de campo | `edge-connector` | Variables, estados, alarmas nativas, calidad y timestamp de origen | Polling o suscripción según protocolo de campo | Modbus / OPC UA / MQTT / OPC DA [fuente: branches/cliente-renovables/problems/scada-general/problem.md] |
| `edge-connector` | `edge-historian` | Telemetría canónica técnica y eventos de proceso locales | Escritura síncrona/batch local | SQL / driver PostgreSQL [est.] [verificar antes de decidir] |
| `edge-connector` | `edge-alarm-engine` | Stream interno de variables normalizadas | Consumo interno en memoria o bus local del proceso | En proceso / canal interno [est.] [verificar antes de decidir] |
| `edge-connector` | `edge-message-gateway` | Telemetría lista para sincronización norte | Publicación asíncrona durable | AMQP [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/outcome.md] |
| `edge-admin-api` | `edge-connector` / `edge-historian` | Configuración, catálogo de señales, health, diagnóstico | Llamadas de administración | REST/HTTPS [est.] [verificar antes de decidir] |
| `edge-message-gateway` | `central-ingestion-gateway` | Lotes/mensajes de telemetría y eventos pendientes | Entrega con ack y reintento | AMQP sobre VPN/WAN [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/outcome.md] |
| `central-ingestion-gateway` | `canonical-ingestor` | Mensajes admitidos desde parques | Consumo de cola interna/bus | AMQP |
| `canonical-ingestor` | `central-historian` | Upserts/insert de telemetría validada y deduplicada | Persistencia de series temporales | SQL / driver PostgreSQL [est.] [verificar antes de decidir] |
| `canonical-ingestor` | `central-alarm-service` | Eventos de dato nuevo, tardío o corregido | Suscripción a stream o consulta incremental | AMQP + consulta TSDB [est.] [verificar antes de decidir] |
| `central-historian` | `reporting-service` | Series temporales, agregados y contexto | Lectura analítica | SQL/API interna |
| `central-historian` + `asset-context-service` | `third-party-api` | Datos temporales enriquecidos y estado de alarmas | Consulta API | REST/HTTPS [fuente: branches/cliente-renovables/problems/scada-general/problem.md] |
| `central-historian` | `ml-raw-export` | Extracciones de histórico crudo/curado | Jobs batch | SQL + exportación a almacenamiento corporativo [est.] [verificar antes de decidir] |
| `central-historian` + `asset-context-service` | `iec61850-publisher` | Variables y estados ya consolidados | Publicación a interfaz de terceros | Adaptador IEC 61850 posterior [fuente: branches/cliente-renovables/problems/scada-general/problem.md] |

## Supuestos
- El catálogo de señales y el canónico técnico mínimo se cerrarán en una fase temprana, porque sin ese contrato la descomposición de servicios pierde valor [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/outcome.md].
- La misma tecnología TSDB en ambos niveles sigue siendo condición del user; esta propuesta asume que la PoC edge de TimescaleDB será satisfactoria [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_a.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/outcome.md].
- La infraestructura de seguridad perimetral, segmentación y publicación a terceros será resuelta por el cliente / su equipo de infraestructura, no por este diseño de aplicación [fuente: branches/cliente-renovables/problems/scada-general/problem.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_a.md].
- La operación local debe seguir siendo posible aunque falle el enlace norte; cualquier control de seguridad o despliegue debe respetar esa restricción de arquitectura [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/final_positions/expert_arquitecto-scada-distribuido.md].

## Preguntas al user
- ¿Queréis que `edge-reporter` exista desde fase 1 como servicio propio, o basta con dejarlo como capacidad embebida en `edge-runtime` hasta cerrar el alcance de informes?
- ¿Preferís presentar al cliente `asset-context-service` como servicio independiente desde el inicio, o como módulo interno de la plataforma central en fase 1 para reducir complejidad percibida?
- ¿La exposición REST a terceros será solo lectura histórica/estado, o también necesitará suscripciones/event push en una fase posterior?

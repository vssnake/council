## Posición

Mi propuesta consolidada final es una arquitectura **cloud-native austera en edge y modular en central**, distinguiendo de forma explícita **deployables reales** de **capacidades internas**.

### 1) Deployables reales vs capacidades internas

#### Edge (deployables reales de fase 1)
| Deployable real | Tipo Kubernetes | Pods recomendados | Responsabilidad |
|---|---|---:|---|
| `scada-edge-runtime` | `Deployment` | 1 pod activo por parque | Adquisición multi-protocolo, normalización técnica, escritura local, alarmado local, reporter local, publicación a outbox y drenaje norte |
| `timescaledb-edge` | `StatefulSet` | 1 pod | Historian local operativo |
| `rabbitmq-edge` | `StatefulSet` | 1 pod | Cola durable local de store-and-forward |

**Capacidades internas de `scada-edge-runtime` (NO deployables separados):** `field-driver-host`, `technical-canonicalizer`, `local-alarm-engine`, `edge-reporter`, `outbox-publisher`, `northbound-uplink`, `edge-ops-api`.

#### Central (deployables reales de fase 1)
| Deployable real | Tipo Kubernetes/OpenShift | Pods recomendados | Responsabilidad |
|---|---|---:|---|
| `rabbitmq-central-ingress` | `StatefulSet` | 2-3 según HA disponible | Punto de entrada AMQPS desde parques |
| `telemetry-ingestion-service` | `Deployment` | 2+ | Consume inbox, valida envelope, deduplica y persiste |
| `timescaledb-central` | `StatefulSet` | 1 primario + réplica según plataforma | Historian corporativo |
| `asset-registry` | `Deployment` | 2 | Catálogo técnico/semántico; **fuera del hot path síncrono de ingestión** |
| `alarm-stream-processor` | `Deployment` | 2 | Procesa stream de alarmas y estados en central |
| `reporting-service` | `Deployment` | 2 | Reporting interno/externo |
| `third-party-api` | `Deployment` | 2 | REST para terceros y frontends |
| `control-plane-api` | `Deployment` | 2 | Configuración, releases y directivas operativas; **auxiliar, no crítico** |

### 2) Runtime y stack propuesto
- **Runtime de aplicaciones**: .NET LTS (`Worker Service` + `ASP.NET Core`).
- **Mensajería**: RabbitMQ con AMQPS.
- **TSDB**: TimescaleDB sobre PostgreSQL en edge y central, sujeto a PoC operativa en edge.
- **Acceso datos .NET**: `Npgsql`.
- **Contratos internos síncronos**: REST interno simple; evitar gRPC en fase 1 salvo necesidad probada.
- **Empaquetado**: imágenes OCI.
- **Despliegue**: Helm charts + Azure DevOps Pipelines.
- **Observabilidad**: OpenTelemetry, métricas Prometheus-compatible, logs estructurados.

### 3) Interfaces inter-servicio que propongo fijar
| Origen | Destino | Contrato | Protocolo | Regla |
|---|---|---|---|---|
| `scada-edge-runtime` | `timescaledb-edge` | muestra normalizada + journal técnico | PostgreSQL | persistencia local |
| `scada-edge-runtime` | `rabbitmq-edge` | evento de telemetría / estado / alarma | AMQP | publicación en outbox local |
| `rabbitmq-edge` / `northbound-uplink` | `rabbitmq-central-ingress` | replay + near-real-time | AMQPS | enlace saliente iniciado desde planta |
| `rabbitmq-central-ingress` | `telemetry-ingestion-service` | inbox central | AMQP | `ack` solo tras persistencia idempotente |
| `telemetry-ingestion-service` | `timescaledb-central` | insert/upsert idempotente | PostgreSQL | persistir primero, enriquecer después |
| `telemetry-ingestion-service` | `asset-registry` | resolución diferida/cacheada | REST interno | **no bloquear ingestión** |
| `timescaledb-central` | `reporting-service` / `third-party-api` | consultas `latest`, `raw-range`, `aggregated-range` | PostgreSQL/REST | superficies de lectura separadas |
| `scada-edge-runtime` | `control-plane-api` | config, manifests, releases | HTTPS REST | edge sigue operando si falla |

### 4) Reglas de plataforma que deben quedar explícitas
- **Edge opera autónomo sin `control-plane-api`**: captura, historiza, alarma localmente y encola backlog.
- **`control-plane-api` es auxiliar**: sirve para configuración y lifecycle, no para operación básica.
- **`asset-registry` no entra en camino síncrono de ingestión**: el dato canónico técnico se acepta y persiste aunque el catálogo no responda.
- **RabbitMQ en edge es mínimo austero**: cola durable simple; no vender `Shovel` sofisticado como base.
- **Telemetría subida incluye tres streams lógicos**: `samples`, `states`, `alarms`.
- **Clave única de muestra fijada**: `park_id + asset_id + signal_id + source_timestamp`.
- **Patrón obligatorio**: `outbox/inbox idempotente` entre edge y central.

### 5) Envelope y versionado
Campos mínimos del envelope norte:
- `schema_version`
- `park_id`
- `asset_id`
- `signal_id`
- `source_timestamp`
- `ingest_timestamp`
- `value`
- `quality_flag`
- `stream_type` (`sample` | `state` | `alarm`)
- `message_id` derivable de la clave única de muestra

### 6) Matriz de compatibilidad edge ↔ central
| Cambio | Compatibilidad requerida | Regla de rollout |
|---|---|---|
| Nuevo campo opcional en envelope | backward compatible | central primero, edge después |
| Nuevo `stream_type` | backward compatible | central debe ignorar/desviar tipos no activados |
| Cambio de semántica de campo existente | breaking | prohibido sin nueva `schema_version` |
| Eliminación de campo | breaking | migración en dos fases |
| Nueva versión Helm edge | debe soportar central N y N+1 | rollout gradual por parques |
| Nueva versión Helm central | debe soportar edge N-1 y N | desplegar antes que edge |

Regla cliente-friendly: **central siempre se despliega antes y mantiene compatibilidad hacia atrás al menos una versión de edge**.

### 7) Helm, manifiestos y pipelines
**Helm charts**
- `charts/scada-edge`: `Deployment` de `scada-edge-runtime`, `StatefulSet` de `timescaledb-edge`, `StatefulSet` de `rabbitmq-edge`, `PVC`, `ConfigMap`, `Secret`, `Service`, `Ingress` solo si aplica.
- `charts/scada-central-platform`: `rabbitmq-central-ingress`, `timescaledb-central`, storage y secretos.
- `charts/scada-central-services`: `telemetry-ingestion-service`, `asset-registry`, `alarm-stream-processor`, `reporting-service`, `third-party-api`, `control-plane-api`.

**Pipelines Azure DevOps**
1. `build-test-dotnet`
2. `build-images`
3. `package-helm`
4. `deploy-central-nonprod`
5. `deploy-central-prod`
6. `publish-edge-bundle`
7. `deploy-edge-ring-by-ring`
8. `rollback-edge` / `rollback-central`

**Manifiestos mínimos**
- `values-edge-<parque>.yaml`
- `values-central-<env>.yaml`
- `NetworkPolicy`
- `PodDisruptionBudget` en central
- `HorizontalPodAutoscaler` solo para servicios stateless centrales
- `PersistentVolumeClaim` explícito para TSDB y RabbitMQ

## Razones

- Responde a la petición del user de **distinguir deployables reales de capacidades internas**.
- Mantiene el edge en el mínimo operativo razonable para un clúster pequeño.
- Evita dos errores de plataforma: dual hot path síncrono y dependencia del catálogo en ingestión.
- Hace explícito que el backbone es **push + store-and-forward + idempotencia**, no REST.
- Añade la **matriz de compatibilidad de versiones** y la estrategia de rollout/rollback que faltaban para una propuesta presentable a cliente.
- Incorpora la decisión ya confirmada por el user: **alarmas también viajan como stream propio**.

## Supuestos

- La infraestructura base de OpenShift/Kubernetes, red y certificados la provee otro equipo.
- TimescaleDB supera la PoC operativa en edge; si no, se revisa solo el motor local, no el patrón arquitectónico.
- La central puede aceptar despliegue previo y convivir con edge N-1/N durante una ventana controlada.
- IEC 61850 queda fuera del backbone de fase 1 y se trata como adaptador posterior.

## Preguntas al user

- Ninguna adicional en esta ronda; con las respuestas recibidas ya puedo cerrar esta posición final.
## Posición

Veo convergencia útil en las tres propuestas: **edge compacto**, **mensajería asíncrona separada de la TSDB** y **central más modular sobre OpenShift**. Mi desacuerdo real no está ahí, sino en que todavía subestiman el problema de plataforma: **qué cabe de verdad en un Kubernetes pequeño en planta**, **qué piezas son compatibles con .NET LTS + OpenShift sin inventar una plataforma excesiva**, y **qué interfaces inter-servicio siguen siendo operables cuando haya backlog de días, upgrades desalineados y soporte remoto limitado**.

Si esto se va a presentar a cliente, no basta con un catálogo de servicios bonito. Hay que cerrar mejor tres cosas: **shape operativo mínimo en planta**, **hot path mínimo en central**, y **contratos entre servicios que no dependan de demasiadas llamadas síncronas ni de dual-writes implícitos**.

## Razones

### Crítica a `proposals/expert_arquitecto-scada-distribuido.md`
- **De acuerdo**: acierta al mantener el límite de **3 unidades desplegables en planta** y al separar el backbone de telemetría del plano REST.
- **Problema concreto**: enumera demasiadas capacidades lógicas en edge (`edge-connector`, `edge-alarm-engine`, `edge-reporter`, `edge-admin-api`) y luego las “colapsa” operativamente. Eso es correcto como narrativa funcional, pero sigue dejando una ambigüedad peligrosa: en un K8s pequeño, si esas capacidades no quedan claramente embebidas en un único `edge-runtime`, el equipo acabará troceándolas de más porque “ya estaban nombradas”.
- **Problema concreto**: `edge-reporter` desde fase 1 me parece débil desde plataforma. Añade superficie HTTP, plantillas, dependencias y troubleshooting en un entorno donde todavía no conocemos ni ancho de banda ni footprint real. Yo no lo vendería como servicio lógico separado aunque luego viva dentro del runtime.
- **Problema concreto**: `central-ingestion-gateway` + `canonical-ingestor` huele a doble salto que no aporta suficiente valor al principio. En OpenShift esto implica más pods, más colas internas, más observabilidad y más puntos de versionado para un flujo que podría resolverse con **broker central + un único `telemetry-ingestion-service`**.
- **Problema concreto**: `asset-context-service` sobre “PostgreSQL/TimescaleDB schema relacional compartido” es una mala frontera de plataforma. Compartir motor o esquema con el historian corporativo facilita el acoplamiento invisible y complica operar backup, tuning y evolución por separado.
- **Qué cambiaría**:
  1. dejar por escrito que en planta solo existen **tres deployables reales**: `edge-runtime`, `timescaledb-edge`, `rabbitmq-edge`;
  2. bajar `edge-reporter` y `edge-admin-api` a capacidades internas del runtime, no a “servicios” presentados como casi independientes;
  3. simplificar la central a `rabbitmq-central` + `telemetry-ingestion-service` + historian + servicios de consumo;
  4. separar `asset-context-service` del motor temporal, al menos a nivel de base de datos y ownership operativo;
  5. añadir una matriz explícita de compatibilidad de versiones edge↔central, porque sin eso el despliegue sincronizado y el rollback quedan en PowerPoint.

### Crítica a `proposals/expert_ingeniero-comunicaciones-industriales.md`
- **De acuerdo**: es la propuesta más sólida en el tramo OT→IT. Acierta con **push saliente desde planta**, **AMQPS**, **RabbitMQ separado de TimescaleDB** y el mínimo canónico técnico.
- **Problema concreto**: confía demasiado en `RabbitMQ Shovel` o mecanismo equivalente sin reconocer su coste operativo en edge. En un clúster pequeño, con soporte remoto y desconexiones largas, meter otra capa de bridging específica puede ser más frágil que un `northbound-publisher` embebido en `edge-runtime` con responsabilidades muy acotadas.
- **Problema concreto**: sugiere `quorum queues` “si la PoC lo soporta”. Yo sería más duro: en planta no asumiría quorum queues como opción por defecto. En tres nodos pequeños, RabbitMQ ya es una pieza cara; si además exigimos replicación fuerte del broker local, podemos terminar gastando demasiados recursos para una función que debe ser austera.
- **Problema concreto**: la propuesta aterriza bien el transporte, pero deja corto el plano de operación. No dice cómo se compatibilizan cambios de esquema del mensaje, cómo se drena backlog tras upgrades, ni cómo se observa de forma simple el estado “capturo local / persisto local / aún no he subido”.
- **Problema concreto**: las interfaces están bien descritas, pero falta un contrato de error operable: `retry`, `dead-letter`, `poison message`, y criterio de `ack` solo tras persistencia correcta en central.
- **Qué cambiaría**:
  1. presentar `RabbitMQ` en edge como **cola durable simple** por defecto; dejar replicación avanzada como validación posterior, no como sugerencia casi estándar;
  2. priorizar un `northbound-publisher` controlado por la aplicación frente a introducir demasiado pronto un mecanismo externo de bridging;
  3. añadir versionado de envelope (`schema_version`) y política de compatibilidad hacia atrás entre edge y central;
  4. fijar explícitamente `ack` después de persistencia idempotente en central y definir `DLQ`/reintentos para mensajes inválidos;
  5. añadir métricas mínimas de plataforma: backlog en cola, edad del mensaje más antiguo, lag de resincronización y estado del uplink.

### Crítica a `proposals/expert_ingeniero-datos-series-temporales.md`
- **De acuerdo**: es la mejor propuesta en disciplina de dato. Acertada la separación entre `raw`, `latest`, `aggregates`, y también la advertencia de no contaminar la hypertable con semántica corporativa mutable.
- **Problema concreto**: desde plataforma, sobredimensiona la central para esta fase. `central-telemetry-ingest`, `telemetry-query-api`, `temporal-retention-jobs`, `signal-catalog-service` y además historian pueden ser razonables a medio plazo, pero para una propuesta inicial al cliente ya rozan el sobre-diseño si no se justifican por ownership claro y despliegue independiente real.
- **Problema concreto**: `signal-catalog-service` en el hot path de ingestión me preocupa. Si `central-telemetry-ingest` necesita consulta síncrona a catálogo para cada muestra o lote, acabamos introduciendo una dependencia que degrada justo el camino crítico que debería ser lo más corto posible en OpenShift.
- **Problema concreto**: su tabla de interfaces dice que `central-telemetry-ingest` consume desde `edge-sync-broker`. Eso, leído literalmente, contradice la topología acordada: la central no debe depender de conectarse al broker del parque. El parque debe **empujar** hacia la central, no exponer su broker como dependencia remota.
- **Problema concreto**: `temporal-retention-jobs` como servicio separado puede ser válido, pero también puede ser burocracia de plataforma. Si esa función no exige ciclo de vida independiente, prefiero resolverla con capacidades nativas de TimescaleDB y automatización acotada, no con otro microservicio permanente.
- **Problema concreto**: habla muy bien del dato, pero poco de OpenShift real: almacenamiento persistente para TimescaleDB, estrategia de upgrade, y aislamiento de recursos entre historian y consumidores. Sin eso, la historia temporal está bien diseñada pero la plataforma sigue difusa.
- **Qué cambiaría**:
  1. sacar `signal-catalog-service` del camino síncrono de ingestión masiva; usarlo para enriquecimiento posterior o cacheado, no como dependencia dura por muestra;
  2. corregir la interfaz para que el parque empuje a un **broker/ingestor central**, nunca al revés;
  3. fusionar al inicio `temporal-retention-jobs` en automatización del propio stack de datos salvo que aparezca un caso claro de despliegue independiente;
  4. reducir el catálogo central inicial al mínimo: ingesta, historian, consultas, alarmado, reporting/API; el resto como evolución;
  5. añadir límites de plataforma: qué servicio puede escalar horizontalmente en OpenShift y cuál requiere cautela por estado/persistencia.

### Lectura transversal del conflicto real
- Las tres propuestas aceptan la compacidad de planta, pero **todavía no blindan suficiente el diseño contra el “despiece accidental”**. Si el documento nombra demasiados servicios edge sin remarcar cuáles son solo capacidades internas, alguien terminará intentando desplegarlos por separado.
- Las tres propuestas aceptan RabbitMQ, pero **ninguna decide con suficiente dureza cuánto broker estamos dispuestos a operar en edge**. Para mí la respuesta debe ser: el mínimo posible.
- Las tres propuestas aceptan OpenShift en central, pero **solo de forma nominal**. Falta aterrizar qué servicios merecen de verdad despliegue independiente y cuáles son solo una separación lógica para el diagrama.
- Ninguna propuesta cierra suficientemente la **compatibilidad de versiones** entre edge y central. Con Azure DevOps, despliegue sincronizado y rollback exigido por el user, esto no es un detalle secundario: es parte de la arquitectura.
- El mayor riesgo de plataforma no es elegir mal entre REST y AMQP; eso ya está bastante resuelto. El mayor riesgo es terminar con **demasiadas piezas vivas** para el tamaño operativo real de los parques.

## Supuestos

- Asumo que en planta el objetivo correcto sigue siendo **tres deployables reales como máximo**.
- Asumo que la comunicación norte debe seguir siendo **push iniciada desde planta**, sin dependencias de conexión desde central hacia los parques.
- Asumo que OpenShift en central sí permite una descomposición moderada, pero no justifica microservicios adicionales si no tienen ownership, escalado y ciclo de vida realmente independientes.
- Asumo que el equipo necesitará convivir con **backlog de días**, **upgrades parciales** y **rollback**, así que los contratos de mensajería deben versionarse desde el principio.

## Preguntas al user

- ¿Queréis que el entregable deje ya explícito qué nombres son **deployables reales** y cuáles son solo **capacidades internas**? Para plataforma, esa distinción cambia mucho la lectura del cliente.
- ¿Preferís que la propuesta presente `RabbitMQ` en edge como **mínimo operativo austero** o queréis dejar abierta explícitamente la puerta a topologías más sofisticadas (`Shovel`, replicación avanzada) sujetas a PoC?
- ¿Queréis que el documento incluya una **matriz de compatibilidad de versiones edge↔central** como parte de la arquitectura, o lo consideráis demasiado detalle para esta fase?

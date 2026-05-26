## Posición

Veo alineación útil en las tres propuestas sobre la dirección general: edge compacto, `push` saliente y `RabbitMQ` separado de la TSDB. Mi crítica es que todavía **no cierran del todo la ingeniería del tramo OT→edge→central**. En las tres aparece el nombre del broker, del historian y de los servicios, pero siguen abiertos puntos que para comunicaciones industriales no son accesorios: **quién es exactamente el terminador de sesión WAN, dónde vive el backlog real, cómo se evita el dual-write local, qué contrato mínimo sale de planta y qué interfaz queda reservada a REST**.

Si el entregable quiere sonar a propuesta técnica seria ante cliente, no basta con decir "AMQP + TimescaleDB". Hay que dejar negro sobre blanco que el backbone es **AMQPS iniciado desde planta**, que el **replay WAN no sale de consultas a la TSDB**, que el canónico técnico sale de planta con identidad y calidad explícitas, y que **IEC 61850 / REST** quedan como capas de publicación posteriores o de consumo, no como partes de la columna vertebral OT.

## Razones

### Crítica a `proposals/expert_arquitecto-scada-distribuido.md`
- **De acuerdo**: acierta al fijar la estrella lógica parque→central, el patrón `push + store-and-forward`, y la idea de un concentrador OT único en planta.
- **Problema concreto**: mezcla "servicios" con "capacidades lógicas" y eso difumina los límites de interfaz. En la tabla aparecen `edge-connector`, `edge-historian`, `edge-message-gateway`, `edge-alarm-engine`, `edge-reporter` y `edge-admin-api`, pero luego se reempaquetan en 3 deployables. Así escrito, no queda claro qué cruza proceso y qué no. Para OT esto importa: cada hop interno extra entre adquisición, buffer y envío añade latencia operativa, superficie de fallo y puntos de desalineación de estado.
- **Problema concreto**: dice que `edge-message-gateway` gestiona confirmación de entrega y replay, pero no aterriza **cómo**: ¿`RabbitMQ Shovel`, publicador .NET, outbox local, o combinación? Decir "RabbitMQ" sin fijar el mecanismo del enlace norte deja un hueco real justo en el punto más sensible del follow-up.
- **Problema concreto**: el contrato de mensaje que propone es demasiado genérico. Habla de `quality` y `source_timestamp`, pero no fija con claridad `message_id`/secuencia técnica estable por muestra. Sin eso, la promesa de deduplicación y orden por señal en central es más retórica que diseño.
- **Problema concreto**: incorpora `iec61850-publisher` y `third-party-api` en el catálogo central con demasiado protagonismo para esta fase. Desde comunicaciones OT eso contamina la narrativa: primero hay que cerrar la columna vertebral de adquisición y sincronización; la publicación a terceros va después.
- **Qué cambiaría**:
  1. fijar explícitamente el mecanismo del enlace `rabbitmq-edge -> rabbitmq-central-ingress`;
  2. declarar que el backlog WAN vive en broker/outbox, no en lecturas desde `edge-historian`;
  3. exigir `message_id` o secuencia técnica obligatoria por muestra;
  4. rebajar IEC 61850 a "adaptador posterior" sin meterlo en el corazón del catálogo.

### Crítica a `proposals/expert_ingeniero-datos-series-temporales.md`
- **De acuerdo**: es la propuesta que mejor define disciplina de dato temporal, separación raw/agregados y la necesidad de no mezclar TSDB con cola de resincronización.
- **Problema concreto**: está bien resuelta como arquitectura de historian, pero le falta bajar a la ingeniería de comunicaciones del enlace edge↔central. Habla de `edge-sync-broker` y `central-telemetry-ingest`, pero deja demasiado abstracto el tramo WAN: quién termina TLS, quién reintenta, dónde se hace el bridging y cómo se evita que el runtime local quede con dos fuentes de verdad.
- **Problema concreto**: propone que `scada-edge-runtime` escribe en `timescale-edge` y publica en `edge-sync-broker`, pero no cierra el patrón de consistencia local. Desde mi ámbito, ahí hay un agujero serio: si falla una de las dos operaciones, ¿quién reconstruye el estado correcto? La propuesta ve bien el problema temporal, pero no cierra la mecánica de comunicaciones que lo soporta.
- **Problema concreto**: la topología central sugiere `edge-sync-broker -> central-telemetry-ingest` pero no nombra un **broker de ingreso en central**. Puede ser una simplificación narrativa, pero técnicamente es demasiado ambigua: si hay varios parques, reintentos, TLS y control de identidad por planta, ese punto merece nombre y responsabilidad propios.
- **Problema concreto**: `signal-catalog-service` se consulta desde `central-telemetry-ingest` por REST interno. Me parece arriesgado si esa consulta entra en el camino crítico de ingestión. Si la ingesta depende síncronamente del catálogo para aceptar cada muestra, acabas acoplando la robustez del enlace OT a una dependencia de control plane.
- **Qué cambiaría**:
  1. introducir explícitamente `central-broker`/`central-ingress-broker` como terminador del enlace norte;
  2. declarar patrón `outbox/inbox` o equivalente para cerrar el dual-write local;
  3. sacar cualquier lookup de catálogo fuera del camino crítico de aceptación de la muestra, dejándolo como caché local o enriquecimiento posterior;
  4. especificar quién inicia, mantiene y supervisa la sesión AMQPS por parque.

### Crítica a `proposals/expert_ingeniero-plataforma-cloud-native.md`
- **De acuerdo**: es la propuesta más completa en catálogo, flujo end-to-end y separación entre plano de datos, control plane y APIs de consumo.
- **Problema concreto**: introduce `scada-control-plane-api` con `poll` HTTPS desde edge para configuración y releases. La idea es compatible con la restricción de red, pero la propuesta no marca una frontera suficientemente dura entre **control plane** y **data plane**. Si no se dice expresamente que la operación de telemetría no depende de ese API, el cliente puede interpretar que el edge necesita hablar con central para operar con normalidad.
- **Problema concreto**: vuelve a aparecer el dual-write: `scada-edge-runtime` persiste en Timescale y publica en RabbitMQ. Falta una garantía explícita de atomicidad lógica o reconciliación. En un sistema OT con cortes WAN, esto no puede quedar implícito.
- **Problema concreto**: mete gRPC para `scada-asset-registry` en central. No tengo objeción al protocolo en sí, pero sí al riesgo de sobreingeniería: en esta fase, lo importante es que el contrato OT hacia arriba sea estable; no veo aún justificado complicar el camino crítico con más interfaces síncronas de las necesarias.
- **Problema concreto**: los endpoints lógicos `alarm-events/<planta-id>` y el uso posible de eventos de alarma derivados en planta pueden crear confusión arquitectónica. El user ha pedido alarmado local y central, sí, pero no necesariamente que el stream de alarmas locales sea un backbone independiente. Tal como está escrito, puede sonar a duplicar semánticas de evento sin cerrar quién es la fuente de verdad.
- **Qué cambiaría**:
  1. dejar explícito que `scada-control-plane-api` es auxiliar y que el edge sigue capturando, alarmando y encolando sin él;
  2. fijar patrón de consistencia local para TSDB + broker;
  3. simplificar dependencias síncronas en el camino de ingestión inicial;
  4. aclarar si las alarmas locales se suben como evento operativo separado o si la fuente de verdad agregada nace solo en central.

### Lectura transversal del conflicto real entre propuestas
- Las tres propuestas aceptan `AMQP/RabbitMQ`, pero **ninguna termina de fijar el mecanismo concreto del puente parque→central**. Para cliente no es lo mismo decir "broker" que decir "broker local + enlace saliente AMQPS gestionado por publisher/shovel + broker de ingreso central".
- Las tres separan TSDB y cola en el discurso, pero **dos dejan implícito un dual-write local** y la del arquitecto deja demasiado abierta la mecánica real del replay. Ese es hoy el mayor gap de coherencia edge↔central.
- Las tres hablan de canónico técnico, pero todavía falta cerrar el mínimo obligatorio del envelope: `park_id`, `asset_id`, `signal_id`, `value`, `unit`, `source_timestamp`, `ingest_timestamp`, `quality_flag`, `message_id` y, si hace falta, secuencia técnica por señal.
- Hay riesgo de contaminar demasiado pronto la propuesta con interfaces de publicación a terceros (`REST`, `IEC 61850`) cuando el problema no resuelto sigue estando en la **columna vertebral OT de adquisición y resincronización**.
- Ninguna propuesta deja del todo claro si el edge puede seguir operando con total normalidad cuando falla el plano de control central. Ese punto debe quedar explícito, porque la restricción de red y de operación aislada no permite ambigüedad.

## Supuestos

- Asumo que el enlace operativo normal para telemetría seguirá siendo **solo saliente desde planta**.
- Asumo que el backlog de varios días obliga a distinguir claramente **historización local** de **buffer de sincronización**.
- Asumo que la deduplicación correcta en central requiere una identidad estable por muestra, no heurísticas temporales.
- Asumo que REST e IEC 61850 son superficies de consumo/publicación, no el backbone OT de esta fase.

## Preguntas al user

- ¿Queréis que el entregable fije ya el **mecanismo concreto** del puente parque→central (por ejemplo `publisher` dentro de `edge-runtime` vs `RabbitMQ Shovel`), o preferís dejarlo como "patrón equivalente"?
- ¿Queréis dejar explícito en el documento que el edge **opera completamente degradado pero autónomo** incluso si falla el `control-plane-api` central?
- ¿La telemetría subida a central debe incluir también **eventos de alarma local** como stream propio, o solo muestras + estados, dejando el alarmado agregado como responsabilidad central?
## Posición

Ninguna de las tres propuestas ajenas está lista para consolidarse tal cual. Comparten la dirección correcta —edge compacto, push saliente, store-and-forward y RabbitMQ separado de la TSDB—, pero discrepan en cortes que para una arquitectura SCADA multinivel son estructurales: **quién termina el canal norte**, **dónde vive el catálogo técnico**, **si la ingesta depende o no de metadatos síncronos** y **qué servicios existen realmente en central desde fase 1**.

Mi lectura es dura: si el lead mezcla estas propuestas sin depurar esas incompatibilidades, saldrá un documento con nombres bonitos pero con interfaces que no encajan entre sí.

## Razones

### Crítica a `expert_ingeniero-plataforma-cloud-native`

**De acuerdo**
- Acierta manteniendo la planta en tres unidades desplegables y reservando la descomposición fina para central.
- Acierta separando TSDB local y broker de sincronización.
- Acierta dejando REST fuera del backbone de telemetría.

**Problemas concretos**
- Mete `scada-asset-registry` en el camino síncrono de `scada-central-ingestor` vía gRPC. Eso es un mal corte: si el catálogo cae o se degrada, la ingestión corporativa queda rehén de un servicio de metadatos. En un sistema distribuido con WAN inestable, primero se debe **persistir el canónico técnico** y luego enriquecer.
- `scada-edge-runtime` concentra demasiadas responsabilidades operativas: adquisición, normalización, alarmado, publicación a cola, drenaje norte y control local. Que vaya en un solo deployable me parece correcto; que arquitectónicamente absorba también el control del replay y del plano de configuración ya no. El límite entre `scada-edge-runtime` y `scada-edge-rabbitmq` queda difuso.
- `scada-control-plane-api` como servicio independiente desde fase 1 me parece prematuro. Añade superficie de fallo y complejidad operacional antes de cerrar el modelo real de operación remota, releases y ownership.
- La central queda quizá demasiado fragmentada para el nivel de incertidumbre actual: broker, ingestor, asset registry, alarm service, reporting, third-party API, IEC adapter y control plane. Eso puede ser una buena foto futura, pero como propuesta base corre el riesgo de vender microservicios antes de cerrar contratos.

**Qué cambiaría**
- Sacaría `scada-asset-registry` del hot path de ingestión: cache local/versionada o enriquecimiento asíncrono posterior.
- Haría explícito un único responsable del replay norte: o un worker lógico dentro de runtime, o un mecanismo broker-to-broker claro, pero no una zona gris entre runtime y RabbitMQ.
- Rebajaría `scada-control-plane-api` a capacidad operativa de fase posterior o a módulo no independiente al inicio.

### Crítica a `expert_ingeniero-comunicaciones-industriales`

**De acuerdo**
- Es la propuesta más clara al fijar que la conexión WAN la inicia planta y que el backlog vive también en planta.
- Acota bien el canónico técnico mínimo que debe salir del edge.
- Acierta al relegar REST a operación/consulta y no a telemetría.

**Problemas concretos**
- Está fuerte en transporte OT, pero débil como arquitectura completa del deliverable. En central deja un catálogo demasiado corto: `rabbitmq-central-ingress`, `telemetry-ingestion-service`, `telemetry-query-api` y `third-party-export-adapter`. Faltan al menos el alarmado agregado, reporting y el servicio de contexto/catálogo que el user ya confirmó como independiente desde fase 1.
- `telemetry-query-api` y `third-party-export-adapter` se pisan. Si ambos existen, no está claro cuál es la fachada oficial hacia consumidores ni dónde termina la responsabilidad de exposición a terceros.
- Apostar ya por `RabbitMQ Shovel` en cada parque me parece arriesgado operativamente. En 25+ sitios remotos, introducir replicación broker-to-broker gestionada por parque puede multiplicar configuración, observabilidad y puntos de fallo. No digo que sea inviable; digo que no lo compraría sin PoC operativa específica.
- La propuesta deja demasiado implícito el catálogo técnico central. Pide un canónico correcto en origen, pero no cierra qué servicio central gobierna identidades, mappings y semántica ampliada. Sin ese ancla, cada parque acaba derivando su propia lectura “canónica”.

**Qué cambiaría**
- Completaría explícitamente el catálogo de servicios centrales de fase 1, no solo la franja de comunicaciones.
- O fusionaría `telemetry-query-api` con la fachada de terceros, o separaría con mucha más precisión consulta interna vs. exposición externa.
- Dejaría `Shovel` como opción `est. [verificar antes de decidir]`, no como sesgo por defecto.
- Añadiría `asset-context-service` independiente desde fase 1, coherente con la respuesta del user.

### Crítica a `expert_ingeniero-datos-series-temporales`

**De acuerdo**
- Es la propuesta más rigurosa en disciplina temporal: `late data`, deduplicación, retención, separación `raw != aggregate`.
- Aporta un contrato temporal más serio que el resto.
- Mantiene la separación correcta entre TSDB y sincronización.

**Problemas concretos**
- Su interfaz principal no cierra arquitectónicamente: `edge-sync-broker -> central-telemetry-ingest` por AMQP omite una pieza explícita en central que termine el canal. Ahí falta un `central-ingestion-gateway` o `central-broker`. Tal como está escrito, el corte edge→central queda ambiguo.
- Se mete demasiado pronto en diseño físico de datos (`telemetry_raw`, `telemetry_gap_log`, `continuous aggregates`, jobs de retención). Para un comité técnico eso es detalle de implementación; en cambio, la topología de servicios queda menos cerrada de lo que debería.
- Repite el error de acoplar ingestión a metadatos: `central-telemetry-ingest` consulta `signal-catalog-service` por REST interno. No acepto que la llegada de telemetría desde 25 sitios dependa de una llamada síncrona a un catálogo por mensaje o por lote.
- Le faltan servicios centrales que el deliverable sí pide como foto funcional: alarmado agregado, reporting y exposición desacoplada a terceros quedan demasiado implícitos o laterales.

**Qué cambiaría**
- Haría explícito un broker/gateway central como terminador del enlace norte.
- Sacaría la consulta al catálogo del camino síncrono de ingestión.
- Movería el detalle de tablas, hypertables y jobs a un anexo de implementación, no al cuerpo principal de arquitectura.

### Inconsistencias entre propuestas que el lead no debería ocultar

- **Misma función, tres nombres**:
  - `edge-historian` / `timescaledb-edge` / `timescale-edge`
  - `edge-message-gateway` / `rabbitmq-edge` / `edge-sync-broker`
  - `central-ingestion-gateway` / `rabbitmq-central-ingress` / *(sin pieza explícita en la propuesta de datos)*
  - `asset-context-service` / `scada-asset-registry` / `signal-catalog-service`
- **Tres modelos distintos para la interfaz con metadatos**: gRPC, REST interno o esquema relacional compartido. No pueden convivir los tres como si fueran equivalentes. Hay que escoger una regla, y mi regla sería: **la ingesta no debe bloquearse por el catálogo**.
- **Tres lecturas distintas del drenaje norte**: un `sync-publisher` dentro del runtime, un `Shovel`/bridge broker-to-broker o un broker edge que aparentemente entrega directamente a un ingestor central. Esas no son variaciones cosméticas; cambian la operabilidad y el ownership del backlog.
- **Nivel de granularidad central inconsistente**: una propuesta vende bounded contexts completos desde fase 1, otra solo describe la franja OT-temporal y otra mezcla arquitectura con detalle de persistencia. El documento final tiene que fijar qué servicios son **fase 1 obligatoria** y cuáles son **capacidades futuras**.

## Supuestos

- Mantengo como línea roja que la planta debe seguir capturando, historizando y alarmando localmente durante días sin WAN.
- Asumo que el entregable es para comité técnico del cliente, no para ingeniería de detalle; por eso rechazo detalle físico de tablas, pero exijo cortes de servicio y contratos entre niveles mucho más nítidos.
- Asumo como cerrado por escalación que `edge-reporter` y `asset-context-service` deben existir como servicios propios desde fase 1.

## Preguntas al user

- Ninguna adicional en esta ronda; las inconsistencias detectadas son internas entre propuestas del panel.

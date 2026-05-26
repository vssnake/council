## Posición

Veo una convergencia útil en las tres propuestas: **planta compacta**, **RabbitMQ separado de la TSDB** y **TimescaleDB como preferente**. Mi desacuerdo no está en esa dirección general, sino en que las tres siguen siendo demasiado blandas en el núcleo temporal: **contrato de escritura**, **contrato de lectura**, **claves de idempotencia**, **gestión de late data** y **frontera exacta entre TSDB y broker**.

Si el entregable quiere ser presentable ante un comité técnico, no basta con nombrar servicios. Tiene que quedar claro **qué se escribe en la TSDB, con qué clave, qué consulta cada consumidor y qué invariantes temporales no se pueden romper**. Hoy esa parte sigue subespecificada.

## Razones

### Crítica a `proposals/expert_arquitecto-scada-distribuido.md`
- **De acuerdo**: acierta al fijar el shape operativo correcto: edge compacto, push saliente, RabbitMQ como store-and-forward y TimescaleDB como historian local y central.
- **Problema concreto**: mezcla demasiado “capacidad lógica” con “servicio” y eso acaba ocultando los contratos temporales reales. Por ejemplo, `edge-historian` aparece como base para persistencia local, consulta operativa e incluso “replay si hay validaciones o relecturas locales”; eso roza otra vez el antipatrón de usar la TSDB como mecanismo parcial de resincronización. El run padre ya dejó que **TSDB != cola de sincronización**.
- **Problema concreto**: propone `edge-historian` con “dato crudo y agregados operativos locales”, pero no delimita si esos agregados son derivados de consulta, continuous aggregates o tablas materializadas separadas. Sin esa distinción, se corre el riesgo de que reporter/alarmado lean proyecciones inconsistentes respecto al raw.
- **Problema concreto**: el `asset-context-service` se apoya en “PostgreSQL/TimescaleDB schema relacional compartido”. No me convence. Si el contexto de activos y la semántica extendida viven en el mismo motor que el historian corporativo, se reabre el acoplamiento entre plano temporal y plano semántico que precisamente queremos evitar.
- **Qué cambiaría**:
  1. dejar explícito que el **replay WAN solo vive en RabbitMQ/outbox**, nunca en consultas a TSDB;
  2. definir una tabla/hypertable raw única y separar de forma explícita las proyecciones de `last value` y agregados;
  3. sacar `asset-context-service` a una base relacional separada, o como mínimo a esquema y patrón de acceso claramente desacoplados del historian;
  4. fijar el contrato de ingesta con claves mínimas: `park_id`, `asset_id`, `signal_id`, `source_timestamp`, `ingest_timestamp`, `quality_flag`, `message_id`, `schema_version`.

### Crítica a `proposals/expert_ingeniero-plataforma-cloud-native.md`
- **De acuerdo**: es la propuesta más ordenada en catálogo de servicios y en separación entre plano de ingestión, consumo y APIs.
- **Problema concreto**: dice que `scada-edge-runtime` persiste en TimescaleDB y “en paralelo” publica en RabbitMQ. Eso, tal como está escrito, deja un agujero serio de consistencia: **dual-write sin coordinación explícita**. Si escribe en TSDB y falla la publicación, o publica y falla la escritura, ¿qué estado es el válido? En una arquitectura temporal con cortes WAN, este punto no es un detalle.
- **Problema concreto**: el `scada-central-ingestor` “deduplica, marca dato tardío y persiste”, pero no define **contra qué clave deduplica** ni dónde mantiene la evidencia de duplicado/reproceso. Sin una clave idempotente estable por muestra, la deduplicación queda en declaración de intenciones.
- **Problema concreto**: propone gRPC para `asset-registry` y JSON canónico para telemetría, pero no aclara la frontera entre metadato de enriquecimiento y dato temporal persistido. Riesgo: enriquecer demasiado en línea y contaminar la hypertable con atributos de alta cardinalidad o de cambio frecuente.
- **Problema concreto**: presenta `scada-central-timescaledb` como base de dashboards, reporting y APIs, pero no baja al patrón de lectura. No es lo mismo servir “último valor por señal” que rangos crudos multi-parque o agregados mensuales. Sin separar `last value`, raw y aggregates, la interfaz de lectura queda coja.
- **Qué cambiaría**:
  1. introducir explícitamente un patrón **outbox/inbox idempotente** entre persistencia local y publicación al broker;
  2. fijar `message_id` o clave natural estable por muestra como requisito obligatorio de ingesta;
  3. definir tres superficies de lectura separadas en central: `latest`, `raw-range`, `aggregated-range`;
  4. prohibir que el enriquecimiento corporativo añada tags/columnas de negocio a la hypertable raw.

### Crítica a `proposals/expert_ingeniero-comunicaciones-industriales.md`
- **De acuerdo**: aterriza mejor que nadie el tramo OT→edge y el hecho decisivo de que la sesión WAN la inicia planta. También acierta al exigir canónico técnico mínimo con timestamp y quality.
- **Problema concreto**: confía demasiado en RabbitMQ Shovel o en el “puente saliente” como si resolver el transporte resolviera también la disciplina temporal. No la resuelve. El broker mueve mensajes; **no define por sí mismo el orden semántico por señal, la idempotencia en central ni la reconciliación de huecos**.
- **Problema concreto**: el canónico propuesto añade `protocol` y `source_endpoint` en cada evento. Como trazabilidad técnica puntual me parece bien; como parte persistente del modelo temporal central me parece peligroso. Son campos candidatos a alta cardinalidad y cambio operativo frecuente; eso puede degradar la TSDB sin aportar valor a la mayoría de queries temporales.
- **Problema concreto**: habla de clave de partición estable por señal para preservar orden, pero no diferencia **orden de entrega en cola** de **orden de persistencia efectivo en TSDB**. Con backlog largo y múltiples consumidores, esa diferencia importa.
- **Problema concreto**: deja poco definida la lectura desde TimescaleDB. Es buena propuesta de comunicaciones, pero insuficiente como propuesta de arquitectura temporal completa porque no dice qué consultas deben optimizarse ni qué vistas/proyecciones existirán.
- **Qué cambiaría**:
  1. mover `protocol` y `source_endpoint` a un journal técnico o tabla auxiliar, no a la hypertable raw principal;
  2. definir ack de consumo solo **después** de persistencia correcta e idempotente en central;
  3. exigir una política explícita de `late arrival` y `gap log`;
  4. complementar la propuesta con contratos de lectura: último valor, rango crudo y agregados.

### Lectura transversal de conflicto entre propuestas
- Las tres propuestas aceptan `source_timestamp` y `quality_flag`, pero **ninguna cierra con suficiente precisión la clave primaria lógica del dato temporal**. Sin eso, hablar de deduplicación, replay correcto y consistencia entre edge y central es prematuro.
- Las tres propuestas nombran TimescaleDB, pero **ninguna baja al esquema mínimo de hypertables/proyecciones**. Echo en falta una decisión explícita: raw append-only + vistas/materializaciones separadas para `last value` y agregados.
- Las tres propuestas separan TSDB y RabbitMQ en el discurso, pero la del arquitecto deja demasiado margen a usar la TSDB como apoyo de replay y la de plataforma deja abierto un dual-write implícito. Ahí hay un conflicto de diseño real que hay que cerrar.
- Ninguna propuesta define suficientemente la **interfaz de lectura**. Para mí este es el mayor hueco del follow-up: se ha mejorado mucho la topología de servicios, pero no todavía la arquitectura de consulta temporal.

## Supuestos

- Asumo que el dato central debe conservar el **timestamp de origen** como eje canónico y tratar `ingest_timestamp` solo como tiempo técnico.
- Asumo que la deduplicación debe ser **determinista por muestra**, no heurística por ventana temporal.
- Asumo que `asset-context-service` y equivalentes no deberían inyectar semántica corporativa mutable dentro de la hypertable raw.
- Asumo que el backlog de días obliga a diseñar explícitamente `late data`, `gap logging` y proyecciones de lectura separadas.

## Preguntas al user

- ¿Queréis que el entregable deje ya fijada la **clave lógica única de una muestra** (por ejemplo `park_id + asset_id + signal_id + source_timestamp [+ secuencia]`), o preferís dejarla como decisión de implementación?
- ¿El histórico central debe priorizar consultas de **último valor**, de **rangos crudos** o de **agregados largos**? Sin ese ranking es fácil diseñar mal los índices/proyecciones.
- ¿Queréis que la trazabilidad técnica (`protocol`, `source_endpoint`, causa de degradación) viva en el historian principal o en un journal técnico separado?
- ¿Aceptáis introducir explícitamente patrón **outbox/inbox idempotente** para cerrar el dual-write edge TSDB + broker, o queréis mantenerlo implícito en la propuesta?
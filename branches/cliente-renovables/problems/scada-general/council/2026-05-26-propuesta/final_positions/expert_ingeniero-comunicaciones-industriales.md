## Posición final
Tras la mediación, sostengo que la arquitectura debe fijar en planta un **concentrador OT** como pieza principal, con salida norte **asíncrona y persistente** apoyada en RabbitMQ, y no tratar la decisión como un simple REST vs broker. La acotación post-debate es esta: acepto que la normalización en planta no sea semántica completa, pero sí debe llegar a un **canónico técnico obligatorio** (`asset/signal id` estable, namespace, unidades/conversiones necesarias, `source timestamp`, `quality flags`) antes de publicar hacia central.

También asumo el compromiso del panel sobre **TimescaleDB como elección preferente única** en planta y central, condicionado a validación operativa real en edge; desde comunicaciones OT eso me parece aceptable precisamente porque el rol de replay y resincronización ordenada queda descargado en RabbitMQ, no en la TSDB.

## Resumen del recorrido
- Posición inicial (Ronda A): defendí un concentrador único de adquisición en planta, contrato canónico técnico hacia arriba y canal saliente con persistencia, replay y semántica explícita de calidad/tiempo de origen.
- Crítica recibida (Ronda B): se me empujó a acotar hasta dónde normalizar en planta y a no sobredimensionar el edge con una semántica completa ni con componentes innecesarios.
- Mediación (Ronda C): cedí en dejar la semántica extendida y de negocio para central; mantuve como irrenunciable la normalización técnica en planta, la separación TSDB/cola de sincronización y el uso de un broker para garantizar orden y tolerancia a desconexión.

## Lo que aporto al moderador
- Punto de acuerdo con el panel: en planta conviene un máximo de 3 unidades desplegables — aplicación de adquisición/procesado, TimescaleDB y RabbitMQ — evitando microservicios finos y preservando un canal `push` desde parque a central.
- Punto de acuerdo con el panel: toda la telemetría debe subir a central conservando `source timestamp` y `quality flags`, informando retraso cuando haya backlog y resincronizando en orden.
- Punto de disenso firme (si lo hay): no mantengo un disenso principal abierto en mi ámbito tras la mediación; mi única cautela es que el compromiso con TimescaleDB no debe interpretarse como cierre definitivo sin PoC operativa real en edge.
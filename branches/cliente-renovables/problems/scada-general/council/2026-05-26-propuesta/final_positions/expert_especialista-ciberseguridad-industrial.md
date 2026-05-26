## Posición final
Tras la mediación, sostengo que la arquitectura debe adoptar **push saliente planta → central, asíncrono, con store-and-forward persistente y RabbitMQ como canal de sincronización**, pero con una acotación no negociable: el enlace norte debe operar con **mTLS obligatorio, identidad máquina-a-máquina, trazabilidad auditable e integridad verificable de los lotes/mensajes**. Acepto el compromiso de **TimescaleDB como elección preferente única** si supera una validación operativa real en edge, y acepto el compromiso de **máximo 3 unidades desplegables en planta** (aplicación de adquisición+procesado, TSDB y RabbitMQ). También acepto el compromiso de un **canónico técnico en planta** y semántica extendida en central.

Mi desacuerdo abierto sigue en el detalle de operabilidad de los controles: la operación local debe sobrevivir a la caída WAN, pero **no debe degradarse el enlace norte a un modo sin autenticación fuerte** por comodidad operativa. Puede haber grace period acotado para certificados y reintento oportunista, pero no un fail-open indefinido hacia central.

## Resumen del recorrido
- Posición inicial (Ronda A): la frontera OT/IT debía modelarse explícitamente con zonas/conduits, mTLS, segmentación mínima efectiva y el módulo único de adquisición tratado como activo crítico.
- Crítica recibida (Ronda B): se me empujó a aterrizar mejor el equilibrio entre controles duros y operabilidad real en edge, evitando exigir mecanismos que bloqueen la operación local o requieran intervención manual frecuente.
- Mediación (Ronda C): cedí en no imponer un vault completo por parque ni controles que dependan de conectividad continua; mantuve como irrenunciable la autenticación mutua, la trazabilidad de acciones remotas, la integridad/deduplicación y la separación de superficies expuestas a terceros.

## Lo que aporto al moderador
- Punto de acuerdo con el panel: convergemos en una planta compacta (máximo 3 unidades), sincronización asíncrona con RabbitMQ, TimescaleDB preferente sujeta a PoC en edge y un contrato canónico técnico saliente desde planta.
- Punto de disenso firme (si lo hay): el moderador debe reflejar que **no respaldo ningún diseño donde la resiliencia operacional justifique perder autenticación fuerte en el enlace planta → central**; la continuidad local puede ser fail-operational, pero la exposición norte no debe pasar a fail-open inseguro.

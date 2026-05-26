## Posición final
Tras la mediación, sostengo una **arquitectura SCADA distribuida en dos niveles**, con **edge autónomo y central desacoplada**, con la siguiente acotación: en planta el diseño debe permanecer **compacto** —sin microservicios finos— y el corte planta → central debe resolverse con **push asíncrono, store-and-forward persistente y RabbitMQ como canal de sincronización separado de la TSDB**. Incorporo el compromiso de panel de **máximo 3 unidades desplegables en planta** (aplicación de adquisición+procesado, TimescaleDB y RabbitMQ), de **canónico técnico en planta** y de **TimescaleDB como elección preferente única** si supera validación operativa real en edge.

Mantengo, no obstante, una reserva abierta: **ningún control transversal debe bloquear la operación local ni exigir intervención manual en planta durante una caída WAN prolongada**. Si el detalle de seguridad contradice ese principio, mi posición sigue siendo priorizar continuidad local con degradación controlada.

## Resumen del recorrido
- Posición inicial (Ronda A): defendí autonomía local fuerte, sincronización asíncrona planta → central, simplicidad operativa en edge y rechazo de REST síncrono como canal primario de telemetría.
- Crítica recibida (Ronda B): se me empujó a aterrizar mejor el contrato de resincronización, a concretar la convivencia con una única TSDB en ambos niveles y a explicitar qué normalización debía hacerse realmente en planta.
- Mediación (Ronda C): cedí en converger hacia **TimescaleDB preferente única** condicionada a PoC en edge, en aceptar **RabbitMQ** como pieza estructural separada y en formalizar un **canónico técnico** en planta; mantuve como línea roja que el edge no debe fragmentarse más de lo necesario y que la operación local no puede quedar rehén de dependencias norte.

## Lo que aporto al moderador
- Punto de acuerdo con el panel: la propuesta debe reflejar una planta compacta, autónoma durante días sin WAN, con sincronización asíncrona ordenada hacia central, semántica técnica mínima común en origen y semántica ampliada en central.
- Punto de disenso firme (si lo hay): el moderador debe reflejar que sigue abierto el detalle de **seguridad vs. operabilidad en edge**; no respaldo un diseño donde la caída o expiración de un control norte impida captura, historización o alarmado local, ni uno que empuje más granularidad de servicios en planta de la estrictamente necesaria.

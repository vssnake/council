## Posición
Valido la hipótesis de arquitectura modular en dos niveles, pero la refino en cuatro puntos: (1) en planta no recomiendo microservicios puros, sino un concentrador local con pocos servicios/módulos desplegables; (2) el corte planta → central debe ser asíncrono y con store-and-forward explícito; (3) HTTP REST encaja para operación, consulta y configuración, pero no como canal primario de telemetría entre planta y central; (4) la central sí puede descomponerse en microservicios o servicios bien separados sobre OpenShift, porque ahí sí existe consumo multicliente, API para terceros, dashboards e integración futura con ML.

## Razones
- La hipótesis de módulo único de adquisición en planta es sólida: reduce el acoplamiento con instrumentación que solo admite una conexión y convierte el parque en un único punto de contrato hacia arriba.
- Con VPN no controlada, ancho de banda desconocido y desconexiones previstas, la autonomía local no es opcional. Si la WAN cae, el parque debe seguir capturando, historizando y alarmando sin depender de la central.
- Por eso, el patrón correcto entre niveles no es llamada síncrona recurrente, sino publicación diferida con reintentos, cola persistente y políticas de prioridad. REST por sí solo no resuelve bien backpressure, reordenación, replay ni desacoplamiento temporal.
- Veo un riesgo claro en empujar la hipótesis de microservicios hasta planta: en un Kubernetes pequeño de tres máquinas por parque, demasiados servicios elevan complejidad operativa, superficie de fallo y coste de mantenimiento distribuido. Aquí el trade-off favorece simplicidad local.
- En cambio, en central sí hay razones estructurales para separar servicios: ingestión, normalización, historización, alarmado agregado, reporting, API REST para terceros y exposición IEC 61850 tienen ritmos de cambio y perfiles de carga distintos.
- También desafiaría implícitamente la idea de subir "todo" a central. En una arquitectura SCADA distribuida, el contrato entre planta y central debe priorizar alarmas, eventos, agregados y series temporales con política explícita; replicar crudo indiscriminado es un Red flag si no se define retención, priorización y downsampling.
- Sobre InfluxDB vs TimescaleDB: no cerraría la decisión solo por preferencia de equipo. Primero hay que fijar si se busca un historiador operativo de borde, un repositorio analítico central o ambos. Mi aportación aquí es que la arquitectura debe tolerar incluso decisiones distintas por nivel si los drivers operativos y analíticos no coinciden.

## Supuestos
- El parque debe seguir operativo durante caídas WAN prolongadas, manteniendo captura, buffer local y alarmado local.
- La latencia planta → central es tolerable siempre que no haya pérdida silenciosa y exista trazabilidad del reenvío.
- El equipo de operación no podrá mantener una malla densa de microservicios heterogéneos en múltiples emplazamientos remotos; conviene minimizar piezas en planta.
- La central consumirá datos ya contextualizados y priorizados, no necesariamente una réplica completa y continua del crudo de campo.

## Preguntas al user
- ¿Las alarmas deben poder reconocerse localmente en planta y luego sincronizar su estado con la central?
- ¿Esperáis enviar a central toda la telemetría de alta frecuencia o aceptáis política de priorización/downsampling por tipo de señal?
- ¿La exigencia de IEC 61850 para terceros aplica desde la primera fase o puede quedar desacoplada en una fase posterior?
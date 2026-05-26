## Respuesta al conflicto `granularidad-servicios-planta` — turno del experto `ingeniero-plataforma-cloud-native`
- **Recibo del lead**: la posición de `arquitecto-scada-distribuido` es que en planta conviene consolidar al máximo porque cada contenedor extra añade complejidad de orquestación, red interna y troubleshooting en un entorno remoto con operación difusa.
- **Mi posición la modifico parcialmente**: no defendería “3 microservicios” por pureza arquitectónica, pero **sí mantengo 2–3 unidades desplegables como máximo** en planta. Mi recomendación concreta pasa a ser: **2 contenedores de aplicación + 1 componente de mensajería persistente solo si RabbitMQ se confirma como pieza explícita**.

### Razón concreta
- **Separaría `adquisición` del resto** porque es el dominio más inestable: drivers, reconexiones OT, timeouts de protocolo y reinicios por periférico. Si cae o se actualiza esa pieza, no quiero tocar la persistencia local ni el backlog ya almacenado.
- **Consolidaría `procesado + alarmado local + TSDB` en una sola unidad** porque ahí sí veo alto acoplamiento funcional y operativo. Separarlos en más servicios generaría chatter interno y debugging distribuido sin beneficio claro.
- **Mantendría `RabbitMQ/sincronización` fuera de la TSDB** porque el propio user ya ha fijado separación entre buffer temporal y cola de resincronización, con requisito de orden. Si la TSDB sufre presión de disco o mantenimiento, la cola debe poder seguir drenando/encolando de forma independiente; y si hay tormenta de replay tras reconexión, no debe arrastrar a la ingestión OT.

### Dónde concedo
- **Concedo** que en edge remoto el criterio dominante no es elegancia cloud-native sino operabilidad. Por eso bajo mi propuesta desde “3 contenedores modulares” a **“máximo 3 piezas, y solo 2 de aplicación”**. No apoyo una malla de servicios finos ni comunicación REST interna entre varios pods en planta.
- **Concedo también** que, si RabbitMQ pudiera ofrecerse como capacidad de plataforma muy estable y externa al ciclo de vida de la app, entonces el diseño visible para el equipo de producto debería sentirse más como **2 módulos operativos** que como 3 servicios a gestionar manualmente.

### Mi posición final
- **No cedo a un contenedor único para todo** salvo prueba fuerte, porque mezcla demasiados modos de fallo: adquisición OT, persistencia temporal, alarmado local y resincronización WAN quedarían acoplados en el mismo proceso y en el mismo ciclo de despliegue.
- **Sí cedo** a reducir la ambición de microservicios: en planta hablaría de **componentes desplegables robustos**, no de microservicios clásicos.
- **Regla práctica**: en cada parque, **no más de 3 pods/servicios de primer nivel atribuibles a la solución SCADA**.

### Lo que necesitaría para mover mi posición
- **Para consolidar más**: evidencia de que un único runtime puede absorber ingestión sostenida, alarmado local, persistencia TSDB y backlog de días **sin bloqueo entre dominios**, con rollback seguro y sin perder datos ni generar alarmas inconsistentes.
- **Para separar más**: evidencia de que hay ownership operativo claro en planta, observabilidad suficiente y beneficios medibles de despliegue independiente que compensen la complejidad añadida. Hoy no la veo.
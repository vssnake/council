## Posición final
Tras la mediación, sostengo una arquitectura **cloud-native híbrida**: en planta, **máximo 3 unidades desplegables** —(1) adquisición + procesado, (2) TimescaleDB, (3) RabbitMQ— y en central una descomposición más fina por bounded contexts sobre OpenShift. La acotación post-debate es esta: acepto **TimescaleDB como elección preferente única** en ambos niveles, pero **solo** si supera una validación operativa real en edge; además, asumo el compromiso de un **canónico técnico** en planta y semántica extendida en central.

Mantengo como irrenunciable que el enlace parque → central sea **asíncrono, persistente y orientado a replay ordenado**; REST puede existir para gestión o control, pero no como backbone de telemetría. También mantengo que, con despliegue sincronizado y ownership operativo difuso en planta, la plataforma debe priorizar **pocas piezas, rollback claro y degradación controlada** antes que pureza de microservicios.

## Resumen del recorrido
- Posición inicial (Ronda A): defendí una arquitectura híbrida, sin microservicios finos en edge, con mensajería persistente para store-and-forward y criterio prudente sobre una única TSDB.
- Crítica recibida (Ronda B): se me empujó a concretar mejor el shape operativo en planta, asumir la restricción de misma tecnología TSDB y no dejar implícita la operación del broker en clusters pequeños.
- Mediación (Ronda C): cedí en converger a **TimescaleDB preferente única** y en explicitar el **canónico técnico** como contrato norte; mantuve como firmes el **máximo de 3 desplegables en planta**, **RabbitMQ separado de la TSDB** y el rechazo a usar REST síncrono como columna vertebral de datos.

## Lo que aporto al moderador
- Punto de acuerdo con el panel: la planta debe ser operacionalmente austera —sin microservicios finos— y publicar toda la telemetría hacia central mediante un canal `push` con persistencia, orden y tolerancia a desconexión.
- Punto de acuerdo con el panel: la propuesta base razonable es **aplicación consolidada + TimescaleDB + RabbitMQ** en planta, con central sobre OpenShift absorbiendo la mayor descomposición funcional.
- Punto de disenso firme (si lo hay): en el desacuerdo seguridad vs. operabilidad edge, sostengo que **ningún control de seguridad debe bloquear la operación local ni exigir intervención manual en planta cuando no haya WAN**; si hay mTLS y gestión de certificados, deben diseñarse con `grace period` y sin convertir expiraciones/remediaciones en causa de caída del servicio local.

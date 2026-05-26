# Resumen del debate — cliente-renovables/scada-general · 2026-05-26-propuesta

## Contexto
El debate se centró en validar una arquitectura SCADA distribuida para un portfolio de generación renovable con un nivel de planta y un nivel central, manteniendo adquisición multi-protocolo, historización temporal, alarmado y capacidades de consumo centralizado para el cliente y terceros [fuente: branches/cliente-renovables/problems/scada-general/problem.md].

El alcance efectivo del panel quedó acotado por varios drivers confirmados durante el debate: autonomía operativa de planta durante días sin WAN, subida de toda la telemetría a central, preferencia por una misma TSDB en ambos niveles, separación explícita entre TSDB y cola de sincronización, despliegue sincronizado entre parques, rollback ante releases defectuosas, y consideración de IEC 61850 como capacidad potencialmente posterior [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/lead_notes.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_a.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_b.md].

## Puntos de acuerdo unánime
- La propuesta debe mantener una arquitectura en dos niveles: edge autónomo en planta y central desacoplada para consumo agregado, reporting y exposición de servicios [fuente: branches/cliente-renovables/problems/scada-general/problem.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/final_positions/expert_arquitecto-scada-distribuido.md].
- En planta no convienen microservicios finos; el consenso converge en un máximo de 3 unidades desplegables: aplicación de adquisición+procesado, TSDB y RabbitMQ [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/debate_mediated.md].
- El enlace planta → central no debe apoyarse en HTTP REST como backbone de telemetría; debe ser asíncrono, persistente y orientado a store-and-forward [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/final_positions/expert_ingeniero-comunicaciones-industriales.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/final_positions/expert_ingeniero-plataforma-cloud-native.md].
- RabbitMQ debe tratarse como componente estructural de sincronización y separado de la TSDB local [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/lead_notes.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_b.md].
- La planta debe emitir un canónico técnico mínimo —IDs estables, namespace, unidades necesarias, source timestamp y quality flags— y reservar la semántica extendida para central [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/debate_mediated.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/final_positions/expert_ingeniero-comunicaciones-industriales.md].
- Toda la telemetría debe subir a central, preservando source timestamp y quality flags, y mostrando explícitamente retraso cuando exista backlog [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/lead_notes.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_a.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_b.md].
- TimescaleDB queda como elección preferente única, pero supeditada a una validación operativa real en edge; si falla esa validación, la decisión debe reabrirse para planta [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/debate_mediated.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/final_positions/expert_ingeniero-datos-series-temporales.md].

## Conflictos debatidos y resolución
### 1. Elección única de TSDB
**Descripción.** El panel debatió si la preferencia del cliente por una sola TSDB debía cerrarse ya o si convenía mantener una bifurcación por nivel.

**Posiciones.**
- Datos temporales y plataforma defendieron converger en una única tecnología por coherencia de operación y mejor encaje con workloads mixtos.
- Comunicaciones y arquitectura aceptaban la convergencia, pero pedían no darla por cerrada sin comprobar su viabilidad real en edge.

**Resolución.** Se resolvió a favor de **TimescaleDB como opción preferente única**, condicionada a una **PoC operativa en edge** antes de consolidarla para producción [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/debate_mediated.md].

### 2. Granularidad de servicios en planta
**Descripción.** Se debatió cuánto desacoplar el edge sin convertirlo en una plataforma difícil de operar.

**Posiciones.**
- Plataforma cloud-native defendía modularidad desplegable corta, pero no microservicios finos.
- Arquitectura SCADA y el arquetipo operacional insistían en consolidar al máximo para reducir blast radius y carga de soporte.

**Resolución.** Se cerró en un **máximo de 3 unidades desplegables en planta**, con la lógica de aplicación consolidada en 1–2 contenedores y RabbitMQ como componente infra separado [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/debate_mediated.md].

### 3. Profundidad de la normalización en planta
**Descripción.** El conflicto era si la planta debía emitir un modelo ya semánticamente rico o limitarse a una normalización técnica robusta.

**Posiciones.**
- Comunicaciones industriales defendía una normalización fuerte en origen para evitar trasladar heterogeneidad aguas arriba.
- Arquitectura SCADA y datos temporales defendían contener la complejidad del edge y dejar el enriquecimiento de negocio en central.

**Resolución.** El panel acordó un **canónico técnico en planta** y **semántica extendida en central** [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/debate_mediated.md].

## Desacuerdos abiertos
### 1. Seguridad fuerte del enlace norte vs. operabilidad edge bajo fallo
**Posición de ciberseguridad OT/IT.** El enlace planta → central debe mantener mTLS obligatorio, identidad máquina-a-máquina, trazabilidad y controles fuertes incluso en escenarios degradados; puede admitirse un grace period acotado, pero no un fail-open indefinido [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/final_positions/expert_especialista-ciberseguridad-industrial.md].

**Posición de arquitectura/plataforma/escepticismo operacional.** Ningún control transversal debe impedir captura, historización o alarmado local, ni exigir intervención manual en planta durante una caída WAN prolongada; si la seguridad operacionalizada bloquea el edge, la arquitectura deja de ser viable [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/final_positions/expert_arquitecto-scada-distribuido.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/final_positions/expert_ingeniero-plataforma-cloud-native.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/final_positions/expert_arquetipo-esceptico-operacional.md].

**Estado.** No hubo cierre porque faltan dos definiciones del cliente: postura de riesgo precisa para fail-open/fail-closed del enlace norte y modelo operativo real en planta, hoy todavía difuso [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/lead_notes.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_b.md].

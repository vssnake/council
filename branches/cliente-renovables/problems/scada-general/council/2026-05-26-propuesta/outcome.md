# Outcome del council — cliente-renovables/scada-general · 2026-05-26-propuesta

## Veredicto sobre la hipótesis del user
La hipótesis del user queda **confirmada en su dirección general** al mantener una arquitectura modular en dos niveles, con edge autónomo y central orientada a consumo agregado, reporting y exposición de servicios [fuente: branches/cliente-renovables/problems/scada-general/problem.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/final_positions/expert_arquitecto-scada-distribuido.md].

Queda **confirmada parcialmente** en la preferencia por microservicios: el debate la valida sobre todo para la central y la corrige en planta hacia una solución compacta, con un máximo de 3 unidades desplegables y sin microservicios finos [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/debate_mediated.md].

Queda **desafiada** en el punto de comunicaciones: HTTP REST no se sostiene como canal primario suficiente para la telemetría planta → central; el consenso lo sustituye por un patrón asíncrono con store-and-forward y RabbitMQ como pieza explícita de sincronización [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/debate_mediated.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/lead_notes.md].

Queda **matizada** en la decisión TSDB: el panel converge hacia **TimescaleDB** como elección preferente única para planta y central, pero únicamente si supera una validación operativa real en edge; por tanto, la hipótesis ya no es “InfluxDB vs. TimescaleDB sin inclinación”, sino “TimescaleDB preferida sujeta a PoC” [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/debate_mediated.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/final_positions/expert_ingeniero-datos-series-temporales.md].

## Contexto, Alcance y Objetivos del Proyecto
El proyecto busca una arquitectura SCADA distribuida para varias plantas solares y parques eólicos, con 3 máquinas por parque para desplegar el sistema de monitorización [fuente: branches/cliente-renovables/problems/scada-general/problem.md].

El alcance funcional confirmado comprende adquisición multi-protocolo, procesado y persistencia de series temporales, alarmado local y central, reporting interno/externo por concretar, y preparación de datos para analítica predictiva futura [fuente: branches/cliente-renovables/problems/scada-general/problem.md].

El objetivo técnico no es solo capturar datos, sino hacerlo de forma que la planta siga operando aunque pierda la WAN durante días, que la central reciba toda la telemetría, y que los peores outcomes —pérdida de datos y alarmas inconsistentes— queden explícitamente mitigados [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/lead_notes.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_a.md].

Queda fuera de alcance el despliegue de infraestructura física, el detalle eléctrico de planta y la definición cerrada de ML; también se mantiene en segundo plano la exposición IEC 61850 a terceros, que puede tratarse como fase posterior [fuente: branches/cliente-renovables/problems/scada-general/problem.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/lead_notes.md].

## Requisitos y Drivers de Arquitectura
Los drivers de arquitectura que más condicionan la solución son los siguientes:
- **Autonomía local real**: la planta debe capturar, historizar y alarmar durante días sin WAN [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_a.md].
- **Flujo norte unidireccional en operación**: la transmisión de datos se inicia desde planta hacia central [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_a.md].
- **Telemetría completa en central**: no se admite recorte funcional del dato a subir; toda la telemetría debe llegar a central [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/lead_notes.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_a.md].
- **Misma TSDB en ambos niveles**: el cliente expresa preferencia por una sola tecnología de series temporales, con sesgo a TimescaleDB [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/lead_notes.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_a.md].
- **Separación entre persistencia y sincronización**: la TSDB local y la cola de resincronización deben ser componentes distintos [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/lead_notes.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_b.md].
- **Operación distribuida con ownership aún difuso**: no hay un equipo de respuesta en planta completamente definido y el estado de hardware/firmware actual sigue siendo incierto [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/lead_notes.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_b.md].
- **Plataforma tecnológica impuesta**: .NET LTS para los módulos, Kubernetes pequeño en planta, OpenShift en central y despliegue por Azure DevOps [fuente: branches/cliente-renovables/problems/scada-general/problem.md].
- **Despliegue coordinado y reversible**: el usuario pidió despliegue sincronizado entre parques y sistema de rollback ante releases malas [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/lead_notes.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_a.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_b.md].

## Visión General de la Arquitectura Propuesta
Se propone una arquitectura de **dos niveles** [fuente: branches/cliente-renovables/problems/scada-general/problem.md]:
- **Nivel planta (edge)**: ejecución compacta y autónoma, con adquisición multi-protocolo, procesado local, TSDB local, alarmado derivado del dato disponible en planta y sincronización asíncrona hacia central [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/final_positions/expert_arquitecto-scada-distribuido.md].
- **Nivel central**: ingestión y consumo agregado sobre OpenShift, con capacidades separadas para historización corporativa, alarmado agregado, reporting, APIs para terceros y evolución futura de capacidades analíticas [fuente: branches/cliente-renovables/problems/scada-general/problem.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/final_positions/expert_ingeniero-plataforma-cloud-native.md].

En planta, el panel converge en un máximo de 3 unidades desplegables: aplicación consolidada de adquisición+procesado, TSDB y RabbitMQ [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/debate_mediated.md]. Este shape busca contener complejidad operacional sin renunciar a store-and-forward, historización local ni separación entre persistencia y resincronización [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/lead_notes.md].

La interfaz entre planta y central debe transportar un **canónico técnico** definido en origen: IDs estables, namespace, unidades/conversiones necesarias, source timestamp y quality flags; la semántica corporativa extendida debe añadirse en central [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/debate_mediated.md].

## Capa de Adquisición de Datos y Comunicaciones Industriales
La adquisición en planta debe materializarse como un **concentrador único multi-protocolo**, coherente con la restricción de equipos que solo soportan una conexión y con la necesidad de centralizar drivers y calidad de dato [fuente: branches/cliente-renovables/problems/scada-general/problem.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/final_positions/expert_ingeniero-comunicaciones-industriales.md].

La normalización en origen no debe intentar resolver toda la semántica del negocio. Debe limitarse al canónico técnico ya acordado, preservando source timestamp y quality flags como campos de primer nivel [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_a.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/debate_mediated.md].

Para el transporte norte, el debate descarta REST como backbone de telemetría. La recomendación es **push saliente desde planta**, con **RabbitMQ** como canal de sincronización persistente y ordenado, separado de la TSDB, dejando HTTP REST para configuración, consulta puntual o exposición API donde corresponda [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/lead_notes.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_b.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/final_positions/expert_ingeniero-plataforma-cloud-native.md].

Dado que el usuario pidió toda la telemetría en central y no fijó priorización durante la resincronización, el sistema debe reinyectar el histórico conforme llega, conservando orden por señal y señalizando retraso cuando el dato llegue tarde [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_b.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/final_positions/expert_ingeniero-datos-series-temporales.md].

IEC 61850 debe tratarse como interfaz de publicación potencialmente posterior, no como condicionante de la columna vertebral de adquisición actual [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/lead_notes.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_a.md].

## Gestión de Datos y Almacenamiento de Series Temporales
La recomendación resultante es **TimescaleDB como TSDB preferente única** para planta y central, pero condicionada a una PoC operativa en edge antes de cerrar la decisión para producción [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/debate_mediated.md].

El papel de la TSDB local no debe confundirse con el del mecanismo de resincronización. La persistencia temporal local y el buffer de subida deben permanecer separados, reservando a RabbitMQ la cola de sincronización y a la TSDB el almacenamiento operativo e histórico local [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/lead_notes.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_b.md].

La disciplina de datos que emerge del debate exige:
- preservación de orden por señal durante replay [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_b.md];
- deduplicación determinista extremo a extremo [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/final_positions/expert_ingeniero-datos-series-temporales.md];
- política de retención y agregación definida desde el inicio, ya que el usuario anticipa agregación histórica con el paso de los años [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_a.md];
- modelado compatible con telemetría completa en central, evitando que la semántica extendida contamine el plano temporal básico [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/debate_mediated.md].

## Plataforma de Microservicios y Despliegue en Contenedores
El panel no recomienda microservicios finos en planta. La propuesta estable es una planta compacta, con un máximo de 3 unidades desplegables y una aplicación de dominio consolidada [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/debate_mediated.md].

En central, sí se justifica una descomposición más fina por bounded contexts, al menos para ingestión corporativa, historización central, alarmado agregado, reporting y exposición a terceros [fuente: branches/cliente-renovables/problems/scada-general/problem.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/final_positions/expert_ingeniero-plataforma-cloud-native.md].

El despliegue debe ser repetible y coordinado entre parques, pero con capacidad explícita de rollback cuando una versión degrade ingestión o alarmado [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_b.md]. La existencia de Azure DevOps como pipeline impuesto y OpenShift como plataforma central encaja con este enfoque, siempre que la estrategia de versiones no convierta un despliegue sincronizado en un fallo simultáneo no reversible [fuente: branches/cliente-renovables/problems/scada-general/problem.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/final_positions/expert_arquetipo-esceptico-operacional.md].

## Seguridad y Cumplimiento Normativo
El consenso del panel fija varios mínimos: patrón push desde planta, identidad máquina-a-máquina, trazabilidad de acciones remotas y separación entre núcleo de ingesta/alarmado y superficies expuestas a terceros [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_a.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/final_positions/expert_especialista-ciberseguridad-industrial.md].

La recomendación de seguridad es, por tanto, modelar el enlace norte con autenticación mutua fuerte y auditoría, y reservar la publicación a terceros (REST e IEC 61850 cuando aplique) a una capa separada del núcleo operativo [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/final_positions/expert_especialista-ciberseguridad-industrial.md].

No obstante, el debate **no cerró** el detalle de fail-open/fail-closed del enlace norte bajo expiración o fallo de controles. La posición de ciberseguridad exige no degradar mTLS de forma abierta; la posición de arquitectura, plataforma y escepticismo operacional exige que ningún control bloquee operación local ni requiera intervención manual frecuente en planta [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/debate_mediated.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/lead_notes.md].

En consecuencia, el entregable debe dejar esta decisión como punto pendiente de definición con el cliente, porque hoy no existe todavía un modelo operativo de respuesta en planta suficientemente claro [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_b.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/lead_notes.md].

## Plan de Implantación, Riesgos y Hoja de Ruta
La secuencia mínima sugerida por el debate es:
1. cerrar contrato de datos y canónico técnico de planta [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/debate_mediated.md];
2. ejecutar PoC operativa de TimescaleDB en edge, con foco en footprint, operabilidad y replay sostenido [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/debate_mediated.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/final_positions/expert_ingeniero-datos-series-temporales.md];
3. validar la topología compacta de planta con RabbitMQ separado de la TSDB [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/lead_notes.md];
4. desplegar con mecanismo de rollback explícito y observación de comportamiento bajo backlog y datos tardíos [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/escalations/round_b.md].

Los riesgos más relevantes identificados por el panel son: ownership operativo difuso en planta, hardware/firmware aún desconocido, viabilidad real de la TSDB elegida en edge, consistencia funcional bajo replay prolongado, y conflicto todavía no resuelto entre seguridad fuerte del enlace norte y operabilidad local bajo degradación [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/lead_notes.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/final_positions/expert_arquetipo-esceptico-operacional.md].

## Anexo: Comparativa Detallada de Tecnologías
### Comparativa TSDB
| Criterio | TimescaleDB | InfluxDB | Lectura del council |
|---|---|---|---|
| Preferencia final del panel | Opción preferente única para planta y central, condicionada a PoC en edge | Alternativa válida si la validación operativa en edge de TimescaleDB falla | El panel converge en TimescaleDB, pero no de forma incondicional [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/debate_mediated.md] |
| Encaje funcional | Mejor alineada con workloads mixtos de series temporales, reporting y consumo corporativo | Mejor percibida como motor temporal especializado y candidato más ligero en edge | El valor diferencial de InfluxDB queda subordinado a que RabbitMQ asume la resincronización [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/proposals/expert_ingeniero-datos-series-temporales.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/debate_mediated.md] |
| Riesgo principal | Debe probar viabilidad operativa real en edge | Introduce divergencia frente a la preferencia del cliente por una sola TSDB | La decisión no se cierra por benchmark aislado, sino por operabilidad multinivel [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/final_positions/expert_ingeniero-datos-series-temporales.md] |

### Comparativa de integración planta → central
| Criterio | HTTP REST como backbone | RabbitMQ + store-and-forward | Lectura del council |
|---|---|---|---|
| Tolerancia a desconexión | Insuficiente como patrón principal para telemetría | Alineado con desconexiones, replay y desacoplamiento temporal | El debate descarta REST como backbone y converge en RabbitMQ como pieza central [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/final_positions/expert_ingeniero-comunicaciones-industriales.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/lead_notes.md] |
| Papel recomendado | Configuración, consulta puntual, APIs | Sincronización persistente planta → central | Ambos pueden coexistir, pero no cumplen el mismo rol [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/proposals/expert_ingeniero-comunicaciones-industriales.md] |

### Comparativa de arquitectura de servicios en planta
| Criterio | Microservicios finos | Planta compacta | Lectura del council |
|---|---|---|---|
| Operabilidad edge | Mayor complejidad y mayor superficie de fallo | Mejor encaje con clusters pequeños y soporte difuso | El panel rechaza microservicios finos en planta [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/debate_mediated.md; fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/final_positions/expert_ingeniero-plataforma-cloud-native.md] |
| Forma recomendada | No recomendada en este alcance | Máximo de 3 unidades desplegables | Es el compromiso operativo del debate [fuente: branches/cliente-renovables/problems/scada-general/council/2026-05-26-propuesta/debate_mediated.md] |

## Refinamientos posteriores

- **`2026-05-26-arquitectura-concreta`** (Tier 2): Detalle concreto de servicios, stack tecnológico, flujos de datos y comunicaciones edge↔central. → `council/2026-05-26-arquitectura-concreta/outcome.md`

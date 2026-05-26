# Lead notes — cliente-renovables/scada-general · 2026-05-26-propuesta

## Directrices del user durante el debate

- **RabbitMQ es importante**: el user pidió explícitamente que RabbitMQ se considere como pieza central del store-and-forward. No es descartable a favor de alternativas (Kafka, NATS, etc.).
- **Toda la telemetría sube al central** — sin excepción. Es requisito de negocio.
- **La planta debe operar días sin WAN** — autonomía operativa real.
- **Se quiere la misma tecnología TSDB en planta y en central** (TimescaleDB preferida).
- **La TSDB y la cola de sincronización son componentes separados** (RabbitMQ NO dentro de TimescaleDB).
- **Despliegue sincronizado entre parques** (no despliegue a medida por parque).
- **Peores outcomes: perder datos + alarmas inconsistentes.**
- **IEC 61850 puede ser fase posterior.**
- **Equipo de operaciones de planta: "difuso, no definido aún"** — el moderador debe reflejar que no hay roles claros de incident responder en campo.
- **Se necesita sistema de rollback para releases malas.**
- **Estado de hardware/firmware: desconocido** — el moderador debe notar que esta incertidumbre afecta a decisiones de edge.

## Intervenciones del lead (mediación)

- **Conflicto tsdb-eleccion-unica**: shuttle entre datos y plataforma. Datos cedió: acepta TimescaleDB única si hay PoC real en edge. Plataforma aceptó la condición del PoC. Compromiso: TimescaleDB como preferente, PoC obligatorio antes de producción.
- **Conflicto granularidad-servicios-planta**: shuttle entre arquitecto y plataforma. Compromiso: máximo 3 desplegables en planta (1-2 contenedores app + RabbitMQ). No microservicios finos en edge.
- **Conflicto normalizacion-profundidad-planta**: shuttle entre comunicaciones y datos. Compromiso: "canónico técnico" en planta (IDs, namespace, timestamps, quality); semántica extendida solo en central.
- **Conflicto seguridad-vs-operabilidad-edge**: shuttle entre ciberseguridad y escéptico. NO resuelto. Ciberseguridad insiste en no degradar mTLS/autenticación. Escéptico exige fail-open para operatividad local. El moderador DEBE reflejar ambas posiciones simétricamente en outcome sin forzar resolución.

## Conflictos abiertos

1. **seguridad-vs-operabilidad-edge**: ¿Qué pasa cuando el certificado expira o la rotación falla y el operador necesita actuar localmente? Ciberseguridad dice fail-closed + breakglass; escéptico dice fail-open con audit. No hay consenso — depende de definición de roles operativos (que el user dice "difusos").

## Contexto fuera de archivos

- El user no tiene definido aún quién respondería a incidentes en planta — esto afecta directamente al conflicto de seguridad vs operabilidad porque el "breakglass" asume alguien con credenciales de emergencia.
- El user desconoce el estado del hardware actual — la viabilidad de TimescaleDB en edge (y del PoC) depende de recursos disponibles que aún no se han inventariado.

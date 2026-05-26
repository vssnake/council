# Debate mediado — cliente-renovables/scada-general · 2026-05-26-propuesta

## Directrices del user (transmitidas al panel)
El user indicó que el uso de un gestor de colas como RabbitMQ puede ser importante para la arquitectura.

## Conflictos resueltos

### `tsdb-eleccion-unica`: TimescaleDB vs. InfluxDB
- **Posiciones iniciales**: datos-TSDB → TimescaleDB en ambos niveles (SQL, ecosistema .NET); comunicaciones → InfluxDB podría ser más ligera en edge.
- **Resolución: compromiso.** Ambos ceden hacia **TimescaleDB como elección preferente única**, condicionada a una **validación operativa explícita en edge** (proof-of-concept con carga real en cluster de 3 nodos). Si la validación falla en footprint/operabilidad, se reconsideraría InfluxDB para planta. El argumento de replay masivo pierde peso porque RabbitMQ absorbe ese rol.

### `granularidad-servicios-planta`: Número de contenedores en planta
- **Posiciones iniciales**: plataforma → 2-3 contenedores modulares; arquitecto → consolidar al máximo (1-2).
- **Resolución: compromiso.** Ambos convergen en **máximo 3 unidades desplegables en planta**: (1) servicio de adquisición + procesado, (2) TSDB (TimescaleDB), (3) RabbitMQ (broker de colas). La lógica de aplicación se consolida en 1-2 contenedores; RabbitMQ queda como componente infra separado. No microservicios finos en edge.

### `normalizacion-profundidad-planta`: Normalización en planta
- **Posiciones iniciales**: comunicaciones → normalización completa canónica en planta; arquitecto → mínima obligatoria, enriquecimiento en central.
- **Resolución: compromiso.** Ambos convergen en un **"canónico técnico" en planta** — IDs estables, namespace, unidades/conversiones necesarias, timestamps de origen, quality flags — y **semántica extendida en central** (jerarquías de assets, metadatos de negocio, taxonomía corporativa). El contrato norte (lo que sale de planta) es un formato canónico técnico definido, no raw pero tampoco full-semantic.

## Conflictos no resueltos

### `seguridad-vs-operabilidad-edge`: Controles de seguridad en planta
- **Posición A** (ciberseguridad): mTLS obligatorio en enlace norte, rotación de certificados tolerante a desconexión (TTL largo, renovación oportunista), trazabilidad local con reenvío diferido, segmentación mínima efectiva. Acepta no imponer vault completo por parque.
- **Posición B** (arquitecto + escéptico): ceden en que hace falta seguridad fuerte, pero mantienen el límite: **ningún control cuya caída bloquee operación local o exija intervención manual en planta**. Si un certificado expira y no hay WAN, el sistema debe seguir operando (grace period).
- **Razón del desacuerdo**: la granularidad del "grace period" y qué componentes exactos deben fallar en modo abierto (fail-open) vs. cerrado (fail-closed) no se puede cerrar sin definir el modelo de amenazas concreto y la postura de riesgo del cliente. Ambas posiciones son compatibles en principio, pero el detalle de implementación (qué falla cómo) queda abierto.

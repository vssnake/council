## Posición
Valido la hipótesis base, pero la desafiaría en dos puntos: **(1)** no tomaría **HTTP REST directo planta ↔ central** como patrón principal de integración OT/IT; lo sustituiría por un **conduit saliente controlado desde planta**, con **store-and-forward**, autenticación mutua y desacoplo de transporte. **(2)** no pondría **microservicios como opción principal en planta**; los reservaría sobre todo para **central**, dejando en parque un conjunto corto de servicios endurecidos y segmentados.

## Razones
- La restricción clave de `problem.md` no es funcional sino de confianza: la central **no debe acceder** a los parques. Eso encaja mejor con un patrón **push desde planta** que con APIs expuestas de forma permanente desde OT.
- La VPN **no sustituye** segmentación ni identidad. Si “todo viaja por la VPN”, pero no hay **mTLS**, control de quién inicia sesión máquina-a-máquina y registro de eventos, el perímetro es aparente, no real.
- El módulo único de adquisición multi-protocolo tiene sentido operativo, pero es también **punto único de compromiso**. Lo trataría como **activo crítico de frontera OT**, aislado del resto de cargas del parque, con allowlist de protocolos/destinos y secretos/certificados gestionados de forma central.
- En planta, más microservicios implican más superficie de ataque: más identidades, más secretos, más políticas de red, más rutas laterales y más complejidad operativa en incidentes. En un entorno con conectividad incierta y soporte remoto limitado, prefiero **menos componentes, más duros**.
- En central sí veo razonable una plataforma de microservicios sobre OpenShift, porque ahí hay más capacidad para segmentación, observabilidad, rotación de secretos, control de ingress/egress y separación entre consumo interno, terceros y analítica.
- La decisión **InfluxDB vs TimescaleDB** no debería cerrarse solo por rendimiento o ergonomía. Desde seguridad, exigiría como mínimos: **TLS en tránsito, cifrado en reposo, RBAC granular, backup/restore probado, auditoría y separación clara entre escritura OT, consumo interno y consumo de terceros**. Si eso no queda resuelto, la elección de TSDB es prematura.
- El canal hacia terceros (REST e IEC 61850) no debe salir de la misma zona lógica que procesa ingesta o alarmas. Requiere **DMZ de publicación** o, como mínimo, separación fuerte de red y de credenciales para evitar que una exposición externa arrastre sistemas núcleo.
- Para el entregable del cliente, propondría explicitar una arquitectura por **zonas y conduits**: campo/dispositivos, servicios OT de parque, buffer/ingesta de parque, DMZ industrial, ingesta central, servicios centrales, zona de publicación a terceros.
- Si el sistema de alarmas solo cubre proceso/dispositivo, se queda corto. Debe incorporar también **señales de ciberseguridad operativa**: pérdida de conectividad anómala, cambio de certificado, reinicios no planificados del gateway, deriva de configuración, intentos fallidos de autenticación y desvíos de tráfico esperados.
- La hipótesis es válida como base comercial/técnica, pero necesita una corrección explícita: **en OT la prioridad no es exponer servicios elegantes, sino reducir caminos de ataque sin romper disponibilidad**.

## Supuestos
- La infraestructura de red, firewalls y segmentación **la implementará otro equipo**, pero aceptará requisitos de zonas/conduits definidos por arquitectura.
- Existe conectividad **saliente** suficiente desde parque hacia central, aunque sea intermitente, para un patrón store-and-forward.
- el cliente aceptará que el acceso de terceros se haga sobre una **capa de publicación separada** y no contra los servicios núcleo del SCADA.
- La gestión de certificados, secretos y rotación no quedará delegada a ficheros locales ni variables de entorno sin gobierno.
- La futura capacidad de ML consumirá datos desde una capa ya saneada/replicada y no directamente desde la frontera OT.

## Preguntas al user
- ¿La restricción “la central no puede acceder a los parques” implica también prohibición de sesiones iniciadas desde central para soporte/mantenimiento, o solo tráfico funcional SCADA?
- ¿Esperáis exponer REST e IEC 61850 a terceros desde la central corporativa, o aceptáis una zona intermedia dedicada de publicación?
- ¿El equipo de infraestructura puede implementar segmentación explícita tipo DMZ industrial y políticas este-oeste entre cargas del parque?
- ¿Hay requisito de trazabilidad/auditoría de acciones remotas sobre los sistemas de parque?

## Crítica a `proposals/expert_arquitecto-scada-distribuido.md`
- **Punto fuerte**: corrige bien el sesgo hacia REST síncrono y reconoce que el enlace planta → central debe ser asíncrono, con store-and-forward y autonomía local.
- **Hueco**: trata la frontera planta/central como problema de resiliencia, pero no como problema de confianza. Falta traducir la arquitectura a **zonas y conduits**, exigir **mTLS**, identidad máquina-a-máquina, segregación de credenciales y una **DMZ industrial** o separación equivalente. Decir “push desde planta” no basta si el concentrador acaba siendo un pivot OT/IT sin controles duros.
- **Hueco**: acepta el módulo único de adquisición como decisión casi solo operativa y no lo trata como **activo crítico de alto impacto**. Si ese nodo cae o se compromete, no se pierde un servicio: se pierde la frontera OT.
- **Supuesto cuestionable**: asume que “priorizar alarmas, eventos y agregados” reduce riesgo sistémico. Con la respuesta del user de que **quieren toda la telemetría en central**, ese recorte ya no es una hipótesis disponible; hay que rediseñar la protección para volumen completo, no para un subconjunto amable.
- **Supuesto cuestionable**: da por hecho que la separación funcional central absorberá bien el riesgo de terceros, pero no especifica cómo aislar REST e IEC 61850 de los servicios núcleo.

## Crítica a `proposals/expert_ingeniero-comunicaciones-industriales.md`
- **Punto fuerte**: separa correctamente el problema de comunicaciones de campo del de integración planta → central y pone sobre la mesa metadatos críticos como `timestamp` de origen y `quality flags`.
- **Hueco**: la propuesta está bien orientada técnicamente, pero sigue siendo demasiado **protocolocéntrica**. No basta con normalizar a OPC UA o a un modelo canónico interno: falta explicitar autenticación mutua, gestión de certificados, control de cipher suites, hardening de endpoints y registro de sesiones/acciones remotas.
- **Hueco**: trata OPC DA y protocolos legacy como cuestión de compatibilidad, no como **deuda de exposición**. Si el mix real de equipos es desconocido, el riesgo no es solo complejidad de drivers; es ampliar superficie con componentes legacy difíciles de securizar y auditar.
- **Hueco**: no aterriza suficientemente el problema de **integridad y replay** en el store-and-forward. Si “lo peor es perder datos y alarmas inconsistentes”, entonces orden, deduplicación, idempotencia y firma/trazabilidad del lote no pueden quedar implícitos.
- **Supuesto cuestionable**: presupone que la VPN permitirá un patrón saliente fiable “aunque intermitente”. Eso puede ser cierto como transporte, pero no demuestra que soporte revocación de certificados, rotación ordenada y observabilidad de fallos sin fricción operacional.

## Crítica a `proposals/expert_ingeniero-datos-series-temporales.md`
- **Punto fuerte**: aterriza bien el sesgo a TimescaleDB para una central analítica y obliga a pensar en retención, agregación y consultas reales en vez de benchmark vacío.
- **Hueco**: la propuesta casi convierte la decisión de TSDB en el eje de la arquitectura cuando, desde ciberseguridad, el problema previo es otro: **cómo separar dominios de escritura OT, consumo interno, reporting y terceros** sin crear un repositorio demasiado expuesto y demasiado privilegiado.
- **Hueco**: al aceptar “misma tecnología en planta y central” como deseable por simplicidad, no cuestiona el riesgo de **vulnerabilidad de modo común**: mismo motor, mismos procedimientos, mismas ventanas de parcheo, mismo patrón de fallo en múltiples emplazamientos más central.
- **Hueco**: no explicita controles mínimos de seguridad del dato histórico: cifrado en tránsito y reposo, RBAC por rol/caso de uso, backup/restore probado, auditoría de accesos y segregación entre datasets internos y exposición a terceros.
- **Supuesto cuestionable**: asume que la normalización de identificadores y el control de cardinalidad se mantendrán disciplinados en el tiempo porque “nuestros módulos” lo harán. Eso resuelve semántica, pero no evita por sí mismo errores de aislamiento, abuso de permisos o contaminación entre dominios.

## Crítica a `proposals/expert_ingeniero-plataforma-cloud-native.md`
- **Punto fuerte**: la propuesta híbrida es razonable y acierta al no empujar microservicios finos a planta.
- **Hueco**: confunde parcialmente “patrón operativo sano en cloud” con “patrón seguro en OT distribuido”. GitOps, OpenShift y mensajería ayudan, pero no sustituyen requisitos de **segmentación**, control de egress, minimización de privilegios, rotación de secretos y trazabilidad fuerte.
- **Hueco**: introducir broker persistente porque “no hay restricciones” no equivale a introducirlo de forma segura. Falta quién custodia credenciales, cómo se segmenta por parque, cómo se evita que un emisor comprometido inunde el backbone y cómo se aísla el canal de terceros del canal troncal.
- **Hueco**: con despliegue “sincronizado” en todos los parques, la propuesta no denuncia el riesgo de **fallo simultáneo a escala**. Desde seguridad/operación, sincronizado no debería significar “big bang”; debería significar versión objetivo común con anillos, validación y rollback.
- **Supuesto cuestionable**: sugiere homogeneidad de plataforma como ventaja neta, pero no pondera que esa homogeneidad también homogeneiza el impacto de una mala configuración, un secreto filtrado o una dependencia vulnerable.

## Crítica a `proposals/expert_arquetipo-esceptico-operacional.md`
- **Punto fuerte**: presiona donde duele: degradación prolongada, buffers saturados, soporte real en edge y comportamiento bajo fallo. Esa fricción es útil y evita arquitectura de brochure.
- **Hueco**: se queda corto en proponer **controles**. Señala bien los riesgos, pero casi no traduce esos riesgos a requisitos concretos de seguridad: zonas/conduits, identidad fuerte, logging remoto, cuentas break-glass, vault de secretos, hardening del nodo crítico, etc.
- **Hueco**: cuestiona Kubernetes y la malla de servicios, pero no distingue suficientemente entre el riesgo de planta y la necesidad de separar en central las superficies expuestas a terceros. Si todo se reduce a “menos piezas”, puedes terminar con menos piezas pero peor aisladas.
- **Hueco**: enfatiza disco, replay y verdad operacional, pero no menciona el problema de **auditoría de acciones remotas**, que el user sí confirmó como requisito.
- **Supuesto cuestionable**: asume que el desconocimiento de hardware/firmware legacy debe llevarnos a frenar la propuesta. Mi lectura es distinta: debe llevarnos a imponer una frontera OT mucho más estricta y un patrón de exposición mínima, no necesariamente a bloquear el diseño modular.

## Posición
Mi posición original se mantiene, pero la endurecería en cuatro puntos:
- el enlace **planta → central** debe ser **saliente, asíncrono, con store-and-forward persistente**, y con garantías explícitas de autenticación mutua, integridad, deduplicación y trazabilidad;
- el **módulo único de adquisición** debe tratarse como **activo crítico de frontera OT**, con aislamiento fuerte y operación mínima, no como simple adaptador funcional;
- la decisión de **misma TSDB** en planta y central solo es aceptable si se acompaña de un baseline claro de hardening, RBAC, backup/restore y parcheo que no introduzca riesgo de modo común inasumible;
- la exposición a **terceros** (REST/IEC 61850) debe quedar separada del núcleo de ingesta y alarmado, aunque la infraestructura concreta la resuelva otro equipo.

## Razones
- El user ya fijó tres restricciones que cambian el análisis: **toda la telemetría va a central**, la **transmisión de datos la inicia el parque** y **perder datos / alarmas inconsistentes** es lo peor.
- Con esos drivers, la discusión ya no es “REST vs cola” en abstracto, sino **cómo evitar pérdida, duplicidad e inconsistencia sin abrir demasiado la frontera OT/IT**.
- Varias propuestas mejoran resiliencia y operabilidad, pero todavía infravaloran la necesidad de modelar explícitamente **quién puede hablar con quién, con qué identidad y con qué evidencia auditable**.

## Supuestos
- el cliente aceptará controles de identidad máquina-a-máquina y segregación de credenciales aunque la infraestructura la ejecute otro equipo.
- La auditoría de acciones remotas deberá cubrir tanto accesos humanos como operaciones relevantes entre componentes.
- El requisito de misma TSDB no elimina la necesidad de separar roles, rutas de acceso y políticas de retención por zona/caso de uso.
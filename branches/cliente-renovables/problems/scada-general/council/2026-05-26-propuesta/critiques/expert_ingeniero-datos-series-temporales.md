## Crítica a `proposals/expert_arquitecto-scada-distribuido.md`
- **Punto fuerte**: acierta al separar claramente edge y central, y al rechazar REST síncrono como patrón primario de telemetría.
- **Hueco**: deja abierta la posibilidad de usar tecnologías distintas de TSDB por nivel, pero el user ya ha dicho que quiere **misma tecnología** en planta y central. Esa bifurcación añade coste operativo justo donde luego habrá que convivir años con downsampling y replay.
- **Hueco**: habla de priorizar agregados y alarmas en el contrato planta → central, pero el user ya ha fijado que quiere **toda la telemetría a central**. No confronta suficientemente el impacto real de esa decisión sobre almacenamiento, cardinalidad, backfill y ventanas de retención.
- **Supuesto cuestionable**: asume que el dato central puede llegar ya “contextualizado y priorizado”; yo veo más probable que llegue crudo-normalizado con metadatos mínimos y que la central tenga que soportar reconciliación y agregación posteriores.
- **Crítica de fondo**: la propuesta es buena a nivel de arquitectura lógica, pero demasiado indulgente con el problema de volumen. Sin orden de magnitud de tags activos, frecuencia efectiva por señal y horizonte de retención cruda, la arquitectura sigue estando subespecificada.

## Crítica a `proposals/expert_ingeniero-comunicaciones-industriales.md`
- **Punto fuerte**: acierta al exigir timestamp de origen, quality flag y patrón push desde planta; eso es imprescindible.
- **Hueco**: se centra en la normalización de protocolos, pero no aterriza cómo se preserva la semántica temporal en retransmisiones largas: orden por partición, watermark de retraso, deduplicación por clave temporal y tratamiento de late arrivals.
- **Hueco**: propone cola persistente porque REST reimplementa una cola “ad hoc”, pero no dice dónde queda el **buffer de verdad** cuando hay días de corte: ¿en disco del concentrador, en la TSDB local, en una outbox aparte? Esa decisión cambia radicalmente la operativa.
- **Supuesto cuestionable**: sugiere OPC UA como interfaz preferente de normalización si existe en origen. Eso puede ser correcto en comunicaciones, pero desde datos no me basta: una normalización demasiado pegada al protocolo puede contaminar el modelo temporal con jerarquías y naming propios del fabricante.
- **Crítica de fondo**: le falta disciplina de modelo de datos. Resolver protocolo no resuelve cardinalidad, retención ni agregación multinivel; y ahí es donde suelen fracasar estos sistemas a los 12-18 meses.

## Crítica a `proposals/expert_ingeniero-plataforma-cloud-native.md`
- **Punto fuerte**: acierta al defender arquitectura híbrida y a reservar la descomposición fina para central.
- **Hueco**: su sesgo hacia TimescaleDB está demasiado apoyado en conveniencia de plataforma y coexistencia con SQL. Eso es útil, pero insuficiente. La elección de TSDB no puede decidirse por ergonomía del ecosistema si luego el patrón dominante es ingestión sostenida + replay masivo + agregados temporales pesados.
- **Hueco**: dice que evitaría dos tecnologías salvo beneficio claro, lo cual suena prudente, pero no confronta la consecuencia: una sola tecnología obliga a optimizar simultáneamente edge y central, que tienen perfiles muy distintos. Necesita defender mejor por qué una misma TSDB soportaría bien ambos extremos.
- **Supuesto cuestionable**: presupone que GitOps/promoción declarativa resuelve gran parte del problema operativo en múltiples emplazamientos. No; eso ayuda al despliegue, pero no resuelve disco caliente, compactaciones, backlog de reenvío ni degradación de queries locales.
- **Crítica de fondo**: la propuesta está demasiado orientada a operabilidad de plataforma y no suficiente a operabilidad del dato. El cuello de botella aquí no será solo OpenShift o Kubernetes; será cómo envejece el histórico temporal sin disparar coste ni degradar consulta.

## Crítica a `proposals/expert_especialista-ciberseguridad-industrial.md`
- **Punto fuerte**: acierta al rechazar acceso iniciado desde central y a exigir separación de zonas y credenciales.
- **Hueco**: la propuesta trata la TSDB casi solo como un componente que debe cumplir TLS/RBAC/backup. Desde datos temporales eso es demasiado superficial: falta discutir cómo asegurar integridad semántica del histórico ante replay, duplicados, datos tardíos y reconciliaciones post-corte.
- **Hueco**: pide una capa de publicación separada para terceros, correcto, pero no valora el impacto sobre duplicación de pipelines de datos, retención adicional y posible divergencia entre dato operativo y dato publicado.
- **Supuesto cuestionable**: asume que el equipo de infra absorberá sin fricción DMZ, segmentación y gestión de secretos. El user precisamente ha quitado eso del foco; arquitectónicamente no podemos basar la solución de datos en que otra capa resolverá cualquier complejidad que le transfiramos.
- **Crítica de fondo**: endurece bien la superficie de ataque, pero no aborda un riesgo igual de serio para el negocio: una arquitectura “segura” que luego no pueda reinyectar históricos consistentes ni sostener consultas multi-parque tras años de acumulación.

## Crítica a `proposals/expert_arquetipo-esceptico-operacional.md`
- **Punto fuerte**: hace la mejor presión sobre fallo, saturación, certificados, relojes y operación degradada. Esa incomodidad es útil.
- **Hueco**: es muy bueno destruyendo supuestos, pero ofrece poca dirección técnica concreta para resolverlos. Si todo queda en “esto puede romperse”, no ayuda a decidir entre alternativas reales.
- **Hueco**: cuestiona la comparativa InfluxDB vs TimescaleDB por académica, pero no propone criterios mínimos de decisión. Precisamente en este caso hacen falta: tasa sostenida de escritura, cardinalidad esperada, patrón de query, política de retención y coste de backfill.
- **Supuesto cuestionable**: trata Kubernetes pequeño en planta casi como red flag en sí mismo. Yo no compro esa conclusión automática; el problema no es Kubernetes per se, sino desplegar demasiadas piezas y no diseñar la persistencia local con disciplina temporal.
- **Crítica de fondo**: su escepticismo operativo es sano, pero le falta separar lo que es riesgo gestionable con buen diseño de datos de lo que es complejidad intrínseca. Si no, se corre el riesgo de recomendar simplicidad aparente que luego sacrifica trazabilidad o capacidad de reenvío correcto.

## Posición ajustada (si cambia)
Mi posición original se mantiene, pero la endurece la ronda: dado que el user ha fijado **misma TSDB en planta y central**, **toda la telemetría a central** y autonomía de **días** sin WAN, la decisión crítica ya no es solo InfluxDB vs TimescaleDB, sino si el diseño soporta tres cosas a la vez: **buffer local duradero**, **replay masivo sin duplicación** y **retención/agregación multinivel**. Cualquier propuesta que no aterrice esos tres mecanismos está, para mí, incompleta.

## Preguntas al user
- ¿Cuál es el orden de magnitud esperado de señales activas por parque y cuántas de ellas van realmente a 1s de forma sostenida?
- ¿Durante el replay tras días sin WAN debéis preservar orden estricto por señal, o basta consistencia eventual con deduplicación?
- ¿Queréis que la TSDB local actúe también como buffer de reenvío, o preferís separar persistencia temporal operativa y cola/outbox de sincronización?
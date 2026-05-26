## Crítica a `proposals/expert_arquitecto-scada-distribuido.md`
- **Punto fuerte**: separa bien planta y central y no romantiza REST para telemetría.
- **Hueco**: sigue asumiendo que la autonomía local “aguantará días” sin aterrizar qué pasa el día 3 cuando el buffer local empieza a competir con alarmas, historización y replay futuro.
- **Hueco**: cuestiona microservicios en planta, pero no entra en el fallo concreto de operar Kubernetes pequeño en sitios remotos: CrashLoopBackOff, volúmenes degradados, relojes desalineados, rotación de certificados.
- **Supuesto cuestionable**: da casi por hecho que el concentrador único puede absorber heterogeneidad de protocolos sin convertirse en cuello de botella técnico y operativo.
- **Pregunta incómoda**: cuando ese concentrador único falle, ¿qué parte del parque sigue viva y cuál queda ciega?

## Crítica a `proposals/expert_ingeniero-comunicaciones-industriales.md`
- **Punto fuerte**: separa correctamente planta ↔ central de comunicación entre servicios y pone el foco en metadatos de origen.
- **Hueco**: se queda demasiado en la capa de transporte. El problema no es solo reenviar; es qué ocurre con backlog, prioridades y reconciliación cuando el user ya ha dicho que lo peor es perder datos y tener alarmas inconsistentes.
- **Hueco**: descansa demasiado en que la planta inicia la comunicación. Eso resuelve dirección del flujo, no operabilidad cuando hay firmware raro, drivers colgados o timestamps corruptos.
- **Supuesto cuestionable**: trata la conservación de `timestamp` y `quality flag` como si por sí sola protegiera de datos tardíos, duplicados o desordenados. No lo hace.
- **Pregunta incómoda**: ¿qué pasa cuando dos horas de backlog llegan de golpe y contaminan alarmas, dashboards y reporter con orden temporal ambiguo?

## Crítica a `proposals/expert_ingeniero-datos-series-temporales.md`
- **Punto fuerte**: aterriza bien la decisión TimescaleDB vs InfluxDB en patrón de consulta y explotación central.
- **Hueco**: sigue siendo una discusión demasiado limpia de base de datos para un entorno que va a recibir toda la telemetría, con reconexiones, replay y posibles ráfagas masivas desde varios parques.
- **Hueco**: acepta “misma tecnología en planta y central” sin pinchar suficiente el coste operacional de esa simetría cuando las condiciones de edge y central no se parecen en nada.
- **Supuesto cuestionable**: presupone que la normalización de señales realmente llegará limpia y estable entre parques. El user dice que la harán los módulos; eso no equivale a que no falle.
- **Pregunta incómoda**: ¿qué pasa cuando la TSDB local entra en presión de disco o corrupción tras apagado brusco? La comparativa tecnológica sirve de poco si la recuperación operativa no está clara.

## Crítica a `proposals/expert_ingeniero-plataforma-cloud-native.md`
- **Punto fuerte**: baja a tierra el riesgo de microservicios finos en planta y reconoce mejor que otros el coste operativo del edge.
- **Hueco**: sigue dando demasiado por sentado que “despliegue sincronizado” y mensajería permitida son buenas noticias. Desplegar sincronizado en ~25 sitios también sincroniza fallos.
- **Hueco**: introduce disciplina de plataforma, pero no pincha lo bastante quién arregla el parque cuando la automatización falla y no hay acceso cómodo ni manos expertas cerca.
- **Supuesto cuestionable**: confía demasiado en que OpenShift ordenado en central compensará la fragilidad potencial del edge. No la compensa; solo la esconde.
- **Pregunta incómoda**: ¿qué pasa el día en que una release mala entra a todos los parques a la vez y la WAN está inestable para revertir?

## Crítica a `proposals/expert_especialista-ciberseguridad-industrial.md`
- **Punto fuerte**: recuerda que VPN no equivale a seguridad y que el módulo de adquisición es activo crítico.
- **Hueco**: depende demasiado de que “otro equipo” materialice bien DMZ, segmentación, gestión de secretos y trazabilidad. Operacionalmente, eso es una apuesta sobre terceros.
- **Hueco**: la propuesta sube mucho el listón de controles, pero no enfrenta el coste real de mantener certificados, identidades y auditoría en parques con conectividad intermitente y hardware/firmware desconocido.
- **Supuesto cuestionable**: asume que endurecer la frontera no añadirá fragilidad operativa. A veces el certificado expirado te tumba antes que el atacante.
- **Pregunta incómoda**: ¿qué pasa cuando un control de seguridad imprescindible expira o se rompe en un parque aislado un fin de semana?

## Posición ajustada (si cambia)
Mi posición original se endurece: las propuestas mejoran la arquitectura en el papel, pero siguen sin tratar la degradación operativa como escenario principal. Todavía veo demasiada confianza en que buffer, Kubernetes, TSDB, certificados y despliegues coordinados se comportarán bien a la vez en 25 emplazamientos heterogéneos.

## Preguntas al user
- ¿Existe una persona/equipo claramente responsable de incidencias de plataforma en planta (Kubernetes, volúmenes, certificados), o eso queda difuso entre desarrollo, infra y operación local?
- Si una release rompe ingestión o alarmas en todos los parques, ¿preferís parar despliegues y perder visibilidad temporalmente, o seguir con operación degradada aunque haya inconsistencias?
- ¿Qué ventana máxima de datos tardíos seguís considerando aceptable antes de que dashboards/reportes/alarmas dejen de ser fiables?
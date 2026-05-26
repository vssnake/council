# Escalations Round A — 2026-05-26-propuesta

## [arquitecto-scada-distribuido, ingeniero-plataforma-cloud-native, arquetipo-esceptico-operacional]
Pregunta (merged): ¿Cuánto tiempo debe aguantar un parque operando sin WAN? ¿Qué pasa si el almacenamiento local se satura?
Respuesta del user: Días. Lo preparamos para que la saturación no pase.

## [arquitecto-scada-distribuido, arquetipo-esceptico-operacional]
Pregunta: ¿Las alarmas deben poder reconocerse localmente en planta y sincronizar estado con central?
Respuesta del user: Las alarmas locales y remotas son las mismas — se derivan de los datos disponibles en cada zona. No hay reconocimiento cruzado.

## [arquitecto-scada-distribuido, ingeniero-datos-series-temporales]
Pregunta: ¿Enviáis toda la telemetría de alta frecuencia a central, o aceptáis downsampling?
Respuesta del user: Toda la telemetría a central.

## [ingeniero-datos-series-temporales]
Pregunta: ¿Qué política de retención para dato crudo en planta y agregados en central?
Respuesta del user: Con el tiempo (unos años) habrá que ir agregando los datos.

## [ingeniero-datos-series-temporales]
Pregunta: ¿Preferís una sola tecnología TSDB en planta y central?
Respuesta del user: Sí, misma tecnología.

## [ingeniero-plataforma-cloud-native]
Pregunta: ¿La central es más de analítica/reporting/API o de ingestión temporal pura?
Respuesta del user: En la central no hay subestación. [Nota del lead: la central es consumidora/analítica, no generadora de datos de campo.]

## [ingeniero-comunicaciones-industriales, especialista-ciberseguridad-industrial]
Pregunta: ¿La comunicación siempre la inicia la planta, o hay casos en que central conecta activamente al parque?
Respuesta del user: La comunicación de transmisión de datos es desde el parque a la central.

## [arquitecto-scada-distribuido, ingeniero-comunicaciones-industriales]
Pregunta: ¿La exigencia de IEC 61850 para terceros aplica desde la primera fase? ¿Qué naturaleza tiene?
Respuesta del user: Puede ser posterior.

## [ingeniero-comunicaciones-industriales]
Pregunta: ¿Qué porcentaje de equipos habla OPC-UA nativo vs. Modbus / OPC DA / propietario?
Respuesta del user: No sabemos.

## [ingeniero-comunicaciones-industriales]
Pregunta: ¿Se conserva timestamp de origen y quality flags como campos de primer nivel en central?
Respuesta del user: Sí.

## [ingeniero-datos-series-temporales]
Pregunta: ¿El identificador de cada señal/dispositivo ya viene normalizado entre parques?
Respuesta del user: Estará normalizado — se encargan de normalizar los datos nuestros propios módulos.

## [especialista-ciberseguridad-industrial]
Pregunta: ¿El equipo de infra puede implementar DMZ industrial y segmentación este-oeste? ¿REST e IEC 61850 a terceros desde zona dedicada?
Respuesta del user: No nos preocupemos de esto, ya tendremos al equipo del cliente para ello.

## [especialista-ciberseguridad-industrial]
Pregunta: ¿Hay requisito de trazabilidad/auditoría de acciones remotas sobre sistemas de parque?
Respuesta del user: Sí.

## [ingeniero-plataforma-cloud-native]
Pregunta: ¿Despliegue de versiones sincronizado en todos los parques o desfase controlado?
Respuesta del user: Despliegue sincronizado.

## [ingeniero-plataforma-cloud-native]
Pregunta: ¿Hay restricción del equipo de infra sobre introducir brokers de mensajería?
Respuesta del user: No hay restricciones.

## [arquetipo-esceptico-operacional]
Pregunta: ¿Quién resuelve una incidencia en planta que requiera tocar Kubernetes/certificados? ¿Qué nivel de intervención manual es aceptable?
Respuesta del user: No queda clara la pregunta para el user. [Pendiente de clarificación en rondas posteriores.]

## [arquetipo-esceptico-operacional]
Pregunta: ¿Qué es peor para el cliente: perder datos, duplicarlos, entregarlos tarde, o alarmas inconsistentes?
Respuesta del user: Lo peor es perder datos y alarmas inconsistentes.

## [arquetipo-esceptico-operacional]
Pregunta: ¿Hasta qué punto se asume hardware/firmware razonablemente sano en dispositivos que pueden llevar años sin actualizar?
Respuesta del user: Es un problema que desconocemos.

# Follow-up — cliente-renovables/scada-general · 2026-05-26-arquitectura-concreta

Refina: 2026-05-26-propuesta  ·  Ancla: secciones 3 (Visión General), 4 (Adquisición), 5 (Datos), 6 (Plataforma)

## Disparador

El outcome de la deliberación padre es demasiado abstracto en la parte de servicios y arquitectura concreta. El user necesita un nivel de detalle que permita presentar al cliente una propuesta técnica con:

1. **Nombres y responsabilidades de cada servicio/contenedor** — tanto en planta (edge) como en central.
2. **Flujo de datos entre componentes** — diagrama lógico describiendo cómo fluye la telemetría desde el equipo de campo hasta la central, pasando por cada servicio.
3. **Comunicación entre edge y los parques** — topología de red lógica y protocolo entre niveles.
4. **Stack tecnológico completo** — frameworks, librerías principales, protocolos específicos (no solo "TimescaleDB + RabbitMQ").
5. **Interfaces/APIs entre servicios** — a nivel de propuesta (no swagger detallado, sino qué expone cada servicio y cómo lo consume el siguiente).

## Información nueva

- El nivel de detalle es "propuesta para el cliente" — profesional pero no un diseño de implementación con contratos API formales. Suficiente para que el comité técnico entienda qué se va a construir.
- La comunicación edge↔parques (inter-parque y parque↔central) es un punto que el outcome padre no cubrió con suficiente especificidad.

## Inclinación del user

Quiere un documento que un responsable técnico del cliente pueda leer y entender: "estos son los servicios, así se comunican, este es el stack". No un documento académico de patrones.

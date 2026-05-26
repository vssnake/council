# Panel — cliente-renovables/scada-general · 2026-05-26-arquitectura-concreta

## Propósito

Detallar la arquitectura concreta: servicios, stack tecnológico, flujos de datos y comunicaciones edge↔central a un nivel que permita presentar propuesta técnica al cliente.

## Especialistas

- **arquitecto-scada-distribuido** (`role_category: Arquitectura de sistemas SCADA multinivel`) — Topología, catálogo de servicios, flujo end-to-end
- **ingeniero-comunicaciones-industriales** (`role_category: Ingeniería de protocolos y comunicaciones OT`) — Comunicación edge↔central, protocolos concretos, interfaces de adquisición
- **ingeniero-datos-series-temporales** (`role_category: Ingeniería de datos temporales a escala`) — Servicio de persistencia, esquema TSDB, flujo de datos temporal
- **ingeniero-plataforma-cloud-native** (`role_category: Plataforma cloud-native en contenedores`) — Descomposición en contenedores, stack de runtime, interfaces inter-servicio

## Notas del panel

### Heredado de
Run padre: `2026-05-26-propuesta` (panel completo de 6 expertos).

### Podados (y razón)
- **especialista-ciberseguridad-industrial** — no aporta a "nombrar servicios y stack concreto"; el tema de seguridad ya está capturado en el outcome padre.
- **arquetipo-esceptico-operacional** — su rol es cuestionar operabilidad, no proponer arquitectura concreta.

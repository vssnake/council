# Panel — casa-rural/filtrado-aguas-grises · 2026-05-25-diseno-sistema

## Propósito
Diseñar un sistema completo de filtrado natural de aguas grises para reutilización en riego, con almacenamiento prolongado.

## Domina
- Selección de arquitectura de tratamiento natural de aguas grises
- Diseño de etapas de filtrado biológico/físico
- Almacenamiento de agua tratada sin degradación
- Compatibilidad del efluente con riego agrícola

## Fuera de alcance
- Tratamiento de aguas negras (fecales)
- Sistemas industriales/prefabricados
- Normativa y trámites administrativos
- Dimensionado exacto con medidas numéricas (fase conceptual)

## Especialistas

- **ingeniero-tratamiento-natural** (`role_category: ingeniería de tratamiento de aguas por métodos naturales`) — diseño de sistemas de depuración natural, etapas de filtrado, dimensionado conceptual
- **biologo-fitodepuracion** (`role_category: biología de fitodepuración y humedales construidos`) — procesos biológicos de purificación, selección de plantas, ecosistema del filtro
- **agronomo-riego** (`role_category: agronomía de riego y calidad de agua para cultivos`) — requisitos del agua para frutales/huerto/jardín, compatibilidad del efluente
- **tecnico-almacenamiento-hidrico** (`role_category: ingeniería de almacenamiento de agua`) — almacenamiento prolongado sin degradación, prevención de olores y anaerobiosis

## Friction archetypes

- **arquetipo-esceptico-operacional** (`dimension: operacional`) — qué falla en el día a día, carga de mantenimiento, problemas estacionales, modos de fallo

## Notas del panel

### Dueño por variable del entregable
- Arquitectura del sistema → ingeniero-tratamiento-natural
- Etapas de filtrado → ingeniero-tratamiento-natural + biologo-fitodepuracion
- Materiales → ingeniero-tratamiento-natural + biologo-fitodepuracion
- Dimensionado → ingeniero-tratamiento-natural
- Almacenamiento → tecnico-almacenamiento-hidrico
- Mantenimiento → arquetipo-esceptico-operacional + todos

### Dimensiones del panel omitidas (y por qué)
- Se omite `político` porque es una decisión individual sin stakeholders múltiples.
- Se omite `adherencia` porque el user muestra compromiso activo con la autoconstrucción.
- Se omite `económico` porque el presupuesto no está definido como restricción en esta fase.

### Decisiones del user sobre composición
Ninguna — panel aceptado en primera propuesta.

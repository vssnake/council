# Propuesta — Ingeniero de tratamiento natural

## Posición

Recomiendo un sistema de tres etapas en serie, por gravedad, basado en:

1. **Pretratamiento**: trampa de grasas + filtro de sólidos gruesos.
2. **Tratamiento biológico principal**: humedal construido de flujo subsuperficial horizontal (HFSS-H).
3. **Etapa de pulido / acondicionamiento pre-almacenaje**: filtro vertical de arena fina + opción de desinfección UV antes del depósito.

Este diseño cubre el objetivo de reutilización para riego con almacenamiento prolongado sin olores, es artesanal y construible con materiales accesibles en la zona mediterránea.

---

## Razones

### ¿Por qué HFSS horizontal como núcleo?

- Flujo subsuperficial elimina mosquitos, olores y contacto humano con el agua — heurística directa de mi dominio: "flujo subsuperficial minimiza mosquitos y olores vs flujo superficial abierto".
- Con espacio sin limitación, la superficie necesaria no es un problema.
- Clima mediterráneo en esta zona: la evapotranspiración estival alta ayuda a concentrar nutrientes para riego pero NO seca un HFSS bien dimensionado si se calcula el balance hídrico.
- Robustez operativa: sin partes móviles, sin electricidad en operación normal.

### ¿Por qué tres etapas y no dos?

- El almacenamiento prolongado (semanas/meses) exige un efluente con DBO₅ muy baja (<20 mg/L) para evitar anaerobiosis en el tanque. Un solo humedal puede no bajar de 30-40 mg/L consistentemente con aguas de cocina+lavadora.
- La etapa de pulido (filtro de arena) baja DBO₅ residual y sólidos suspendidos, garantizando estabilidad en depósito.

### ¿Por qué UV como opción?

- No es químico, bajo consumo, sin residuos. Previene proliferación bacteriana durante almacenamiento largo.
- Solo necesario si el almacenamiento supera 2-3 semanas — compatible con la hipótesis validada por el user.

---

## Arquitectura del sistema

```
Desagüe aguas grises (casa)
        │  (gravedad)
        ▼
┌─────────────────────┐
│  1. PRETRATAMIENTO  │
│  Trampa grasas +    │
│  filtro grueso      │
└────────┬────────────┘
         │  (gravedad)
         ▼
┌─────────────────────────────────┐
│  2. HUMEDAL SUBSUPERFICIAL      │
│  Horizontal (HFSS-H)           │
│  Grava graduada + plantación   │
└────────┬────────────────────────┘
         │  (gravedad)
         ▼
┌─────────────────────────────────┐
│  3. FILTRO VERTICAL DE PULIDO   │
│  Arena fina / arena-carbón      │
└────────┬────────────────────────┘
         │  (gravedad o bombeo mínimo)
         ▼
┌──────────────────┐
│  [UV opcional]   │
└────────┬─────────┘
         ▼
┌──────────────────────────┐
│  DEPÓSITO DE ALMACENAJE  │
│  (cerrado, opaco)        │
└──────────────────────────┘
         │
         ▼
       RIEGO
```

---

## Etapas de filtrado — detalle

### Etapa 1: Pretratamiento

| Elemento | Función | Material |
|----------|---------|----------|
| Trampa de grasas | Retener grasas/aceites de cocina y detergentes | Arqueta de obra o bidón PE 200 L con deflectores internos |
| Filtro de sólidos gruesos | Retener fibras textiles, restos comida | Cesta extraíble con malla acero inox ~2-3 mm |

- **Mantenimiento**: limpieza de la cesta semanal; vaciado de grasas cada 1-3 meses según carga.
- **Red flag sin pretratamiento**: colmatación del humedal en meses.

### Etapa 2: Humedal construido HFSS-H

| Parámetro | Valor / Rango |
|-----------|---------------|
| Tipo de flujo | Subsuperficial horizontal |
| Medio filtrante | Grava lavada: capa inferior 20-40 mm, capa superior 6-12 mm |
| Profundidad útil | est. 0,5-0,6 m `[verificar antes de decidir]` |
| Superficie | est. 4-6 m² para 2 personas (~200 L/día de aguas grises) `[verificar antes de decidir]` |
| Tiempo de retención hidráulica (TRH) | est. 3-5 días `[verificar antes de decidir]` |
| Pendiente de fondo | 0,5-1% |
| Impermeabilización | Lámina EPDM o PE 1 mm sobre geotextil |
| Plantación | Phragmites australis, Typha latifolia u otras macrófitas locales (fuera de mi limitación — consultar con botánico/agrónomo) |

- Criterio de dimensionado estándar para aguas grises domésticas: est. 2-3 m² por persona equivalente para HFSS-H `[fuente: UN-Habitat, "Constructed Wetlands Manual", 2008, Table 5.2]`.
- Con 2 personas y carga de cocina+lavadora (más alta en DBO que solo duchas), aplico margen: ~3 m²/persona → 6 m² total.

### Etapa 3: Filtro vertical de pulido

| Parámetro | Valor / Rango |
|-----------|---------------|
| Medio | Arena silícea 0,5-2 mm (capa principal 60-80 cm) sobre grava drenante 20-40 mm (capa 15-20 cm) |
| Superficie | est. 1-2 m² `[verificar antes de decidir]` |
| Modo de carga | Intermitente (llenado-vaciado con sifón de descarga) para mantener aerobiosis |
| Función | Reducir DBO₅ residual, SS y patógenos por filtración + biofilm aerobio |

### Desinfección UV (opcional)

- Lámpara UV-C 25-40 W, caudal de paso lento.
- Solo justificada si almacenamiento > 2-3 semanas.
- est. Coste de lámpara: 80-200 € `[verificar antes de decidir]`.
- Consumo: est. ~30-40 W en operación, pocas horas al día (solo cuando hay flujo al depósito).

---

## Materiales principales

| Material | Uso | Accesibilidad en zona mediterránea |
|----------|-----|-------------------------------|
| Grava lavada (varias granulometrías) | Medio filtrante humedal | Cantera local — alta disponibilidad |
| Arena silícea lavada | Filtro de pulido | Proveedor de áridos — alta disponibilidad |
| Lámina EPDM/PE 1 mm | Impermeabilización humedal | Almacenes de construcción / riego |
| Geotextil 200-300 g/m² | Protección de lámina | Almacenes de construcción |
| Bidón PE 200-1000 L | Trampa grasas, arquetas | Reciclaje industrial / agrícola |
| Tubería PVC 75-110 mm | Conducciones, distribución | Fontanería estándar |
| Malla acero inox | Filtro grueso | Ferretería industrial |

---

## Dimensionado orientativo

| Parámetro | Estimación |
|-----------|-----------|
| Caudal diario (2 personas, todas las fuentes grises) | est. 150-250 L/día `[verificar antes de decidir]` |
| Superficie humedal HFSS-H | est. 5-6 m² (con margen para ampliación futura) |
| Superficie filtro pulido | est. 1,5 m² |
| Volumen trampa grasas | est. 100-200 L |
| Superficie total sistema (incluyendo accesos) | est. 12-15 m² |

---

## Almacenamiento

Fuera de mi limitación técnica directa (ver persona.md), pero recomendaciones desde la perspectiva de calidad del efluente que entrego al depósito:

- El efluente post-pulido + UV debería tener DBO₅ < 15-20 mg/L y SS < 20 mg/L — condiciones para almacenamiento sin anaerobiosis.
- Depósito cerrado y opaco (evita algas).
- Ventilación pasiva del depósito para mantener condiciones aerobias.
- Si no se usa UV, considerar recirculación periódica o aireación pasiva en el depósito para evitar estratificación.

---

## Mantenimiento

| Tarea | Frecuencia |
|-------|-----------|
| Vaciar cesta filtro grueso | Semanal |
| Limpiar trampa grasas | Cada 1-3 meses |
| Inspeccionar flujo humedal (que no haya encharcamiento superficial) | Mensual |
| Rastrillar superficie filtro pulido (si colmata) | Cada 3-6 meses |
| Reemplazar arena colmatada filtro pulido (capa superior 5-10 cm) | Cada 1-2 años |
| Revisión/reemplazo lámpara UV | Anual (si se instala) |
| Poda macrófitas humedal | Anual (final de invierno) |

---

## Supuestos

- est. Caudal 150-250 L/día para 2 personas (incluye cocina + lavadora + duchas + lavabos) `[verificar antes de decidir]` — el user debería medir o estimar su consumo real.
- El terreno tiene pendiente natural suficiente (>0,5 m de desnivel disponible entre salida casa y punto final) para flujo por gravedad en todas las etapas. Si no, se necesita un bombeo mínimo.
- Temperatura media del agua en invierno ≥ 10°C (esta zona rara vez baja de 5°C ambiente en invierno) — suficiente para actividad biológica mínima en humedal.
- Las aguas grises NO incluyen inodoros (confirmado en problem.md — solo fregadero, lavadora, lavavajillas, duchas, lavabos).

---

## Preguntas al user

- ¿Cuántos litros/día estimas de consumo de agua en la vivienda? (o bien: ¿cuántas duchas/lavadoras/ciclos de lavavajillas por día?)
- ¿Hay desnivel natural entre la salida de las tuberías de la casa y la zona donde se instalaría el sistema? ¿Aproximadamente cuántos metros de caída?
- ¿Cuánto tiempo máximo prevés almacenar el agua antes de regar? (¿semanas, meses?)
- ¿Hay previsión concreta de ampliación de ocupantes? (¿a 4? ¿a 6?) — para dimensionar con margen adecuado.
- ¿Usáis productos de limpieza/detergentes ecológicos o convencionales? (afecta toxicidad del influente al humedal).

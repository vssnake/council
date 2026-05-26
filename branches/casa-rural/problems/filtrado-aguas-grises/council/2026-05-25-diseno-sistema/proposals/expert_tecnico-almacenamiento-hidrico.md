## Posición

El almacenamiento prolongado (semanas/meses) de agua gris tratada es el punto crítico del sistema descrito en `problem.md`. Valido la hipótesis general, pero **refino con una condición sine qua non**: sin un subsistema de conservación activa o pasiva post-tratamiento, el agua se degradará en ≤10 días independientemente de lo bueno que sea el filtrado previo. Propongo un esquema de almacenamiento en **depósito opaco de PEAD alimentario, enterrado o semi-enterrado, con aireación pasiva y rebosadero obligatorio**, dimensionado para cubrir el desfase estacional producción-consumo típico de esta zona.

---

## Razones

### 1. Heurística central: agua tratada ≠ agua limpia
El efluente de cualquier sistema natural (humedal, biofiltro) conserva carga orgánica residual (DBO₅ típica de salida de humedal construido: 10–30 mg/L — fuente: García & Corzo, *Depuración con Humedales Construidos*, UPC, 2008). Esa carga, en ambiente cerrado sin O₂, genera anaerobiosis → sulfuros → olor. Esto invalida un tanque hermético simple.

### 2. Depósito opaco enterrado/semi-enterrado
- **Oscuridad**: elimina fotosíntesis → sin proliferación de algas.
- **Temperatura estable**: enterrado en esta zona la temperatura del suelo a 1 m ronda 15–18 °C en verano (est. [verificar antes de decidir]), reduciendo actividad bacteriana respecto a superficie (~35 °C).
- **Material**: PEAD alimentario (polietileno de alta densidad) — inerte, no alcaliniza como el hormigón sin tratar, no corroe como el metal. Depósitos de 2000–5000 L son accesibles y no prefabricados-industriales en el sentido rechazado por el user (se consiguen como tanques agrícolas estándar).

### 3. Aireación pasiva (no eléctrica)
- Tubo de ventilación con malla anti-insectos (DN 50–75 mm) en la parte superior — permite intercambio gaseoso sin luz directa.
- Opción mejorada: circuito de convección tipo "chimenea solar" — un tubo negro expuesto al sol crea tiro natural que fuerza renovación de aire sobre la lámina de agua. Coste ≈ tubería PVC + codo, sin bomba.
- Si se quiere más garantía: recirculación por bomba solar pequeña (5–10 W) que mueve agua del fondo a la superficie 1–2 h/día. Esto es "tecnología puntual" compatible con la filosofía del user.

### 4. Dimensionado orientativo

| Parámetro | Valor estimado | Fuente/nota |
|---|---|---|
| Producción aguas grises 2 personas | 100–150 L/día | est. media española ~70 L/pers/día grises (INE, encuesta agua 2020) |
| Pérdida en tratamiento (humedal) | ~20–30% (evaporación + absorción) | García & Corzo 2008 |
| Efluente disponible | ~70–120 L/día | |
| Demanda riego verano (frutales+huerto+jardín) | 200–500 L/día | est. [verificar antes de decidir] — depende de superficie |
| Demanda riego invierno | ~0 L/día (lluvia cubre) | |
| Desfase estacional máximo | Invierno: acumulación ~70–120 L/día × 90 días = 6.300–10.800 L | |
| **Volumen recomendado depósito** | **5.000–10.000 L** | Rango para cubrir 1–3 meses de acumulación invernal |

### 5. Rebosadero obligatorio
En invierno el riego se detiene y la producción de aguas grises continúa. El depósito se llenará. Se necesita un punto de vertido seguro: zanja de infiltración, derivación al terreno arbolado, o conexión al pozo negro existente como overflow.

### 6. Protección contra mosquitos
Toda apertura (ventilación, entrada, rebosadero) debe llevar malla <1 mm. Alternativa: tapa flotante de poliestireno que impide oviposición pero permite intercambio gaseoso perimetral.

---

## Arquitectura de almacenamiento propuesta (tramo post-tratamiento → riego)

```
[Efluente del sistema de filtrado]
         │
         ▼
 ┌──────────────────────────┐
 │  DEPÓSITO PEAD 5000-10000L │  ← Enterrado/semi-enterrado
 │  (opaco, alimentario)      │  ← Ventilación pasiva (tubo+malla)
 │                            │  ← Entrada sumergida (evita salpicadura)
 │                            │  ← Salida con flotador (toma agua superficial, más oxigenada)
 └──────────┬───────────────┘
            │ rebosadero ──→ zanja infiltración / terreno arbolado
            │
            ▼
 [Bomba de riego / gravedad si hay desnivel]
            │
            ▼
 [Frutales / huerto / jardín]
```

### Detalle de componentes

| Componente | Material / especificación | Función |
|---|---|---|
| Depósito principal | PEAD alimentario, 5000 L mínimo | Almacenamiento prolongado sin degradación |
| Ventilación | Tubo PVC DN75, codo 180°, malla inox <1mm | Intercambio O₂, anti-insectos, anti-lluvia |
| Entrada | Tubo sumergido 10–15 cm bajo nivel agua | Evita oxigenación excesiva (turbulencia → resuspensión sedimentos) |
| Toma de riego | Flotador con filtro grueso | Agua más limpia (superficie) |
| Rebosadero | Tubo DN110 a cota máxima −5cm | Derivación excedente |
| Opción UV (tecnología puntual) | Lámpara UV-C inline antes del depósito | Reduce patógenos → almacenamiento más seguro y largo |
| Opción recirculación | Bomba solar 5–10 W + timer | Aireación activa para >30 días retención |

---

## Supuestos

- est. [verificar antes de decidir]: la demanda de riego en verano (superficie concreta de frutales/huerto/jardín no especificada) — esto afecta directamente al dimensionado.
- Asumo que hay desnivel disponible o disposición a usar bomba para riego (no especificado en problem.md).
- Asumo que el sistema de filtrado previo logra DBO₅ <30 mg/L — si la calidad de entrada al depósito es peor, el almacenamiento >2 semanas sin aireación activa no es viable.
- Asumo que "no prefabricado comercial" se refiere a sistemas de tratamiento tipo Biorock/SBR, no a un simple tanque de PEAD agrícola (que es material base, no un sistema cerrado propietario).

---

## Preguntas al user

- ¿Cuánta superficie aproximada de riego tienes (frutales + huerto + jardín)? Esto define si 5000 L bastan o necesitas 10.000+.
- ¿Hay desnivel natural entre la zona del sistema de filtrado y la zona de riego, o necesitarás bombeo?
- ¿Un tanque de PEAD agrícola (tipo los que se usan para almacenar agua de lluvia en fincas) entra dentro de lo aceptable, o lo consideras "prefabricado comercial"?
- ¿Aceptas una bomba solar pequeña (5–10 W) para recirculación, o prefieres 100% pasivo aunque limite el tiempo máximo de almacenamiento sin olor a ~2–3 semanas?

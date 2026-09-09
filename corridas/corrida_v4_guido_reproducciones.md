# Registro — reproducciones del caso Guido (prueba de estabilidad)

**Objetivo:** aplicar el criterio de reproducción del README — mismo hilo, misma fecha
histórica (10/8), mismos "Son 8 niños", mismo modelo (`gemini-3.6-flash`), temperatura 0 —
y verificar que el resultado se mantiene. Referencia: [corrida_v4_guido.md](corrida_v4_guido.md)
(6/9, contrato v4.1): **carta, $503.280, sin bebida** (16 sesiones de 60 + 8 Box Cafe − 10%).

Todas las corridas: frontend `index.html` → API Gemini · operador Mateo García ·
precios `2026-09-06.1` · tokens y hora en cada fila.

## Resultado de la saga (4 corridas del 9/9, madrugada)

| # | Hora | Contrato | Total | ¿Reproduce? | Qué pasó |
|---|---|---|---:|---|---|
| R1 | 02:18:31 | v4.1 | $568.080 | ❌ | Agregó **16 Gaseosas ($72.000) que nadie pidió** — leyó el supuesto "bebida sin cantidad: 2 por persona" como aplicable aun sin mención de bebida. Tokens 6.529/661. |
| R2 | 02:20:51 | v4.1* | $568.080 | ❌ | Corrida inválida como test de v4.2: el navegador sirvió el contrato **cacheado** (mismos 6.529 tokens de entrada). Expuso un bug real del frontend → fix `cache: no-store`. |
| R3 | 02:22:07 | v4.2 | $438.480 | ❌ | Sin bebida ✓ (regla nueva funcionó), pero cotizó cada chico como **una sesión de 120 min ($49.000)** en vez de dos de 60 ($58.000) — la ambigüedad de la iteración 2 de la Entrega 2, reaparecida. Tokens 6.604/563. |
| R4 | 02:23:57 | v4.3 | **$503.280** | ✅ | 16 sesiones de 60, Box Cafe ×8, sin bebida, completo, 8/8 checks. Tokens 6.728/588. |
| R5 | 02:24:50 | v4.3 | **$503.280** | ✅ | Idéntica en modo, totales, desglose y estado; solo varía la redacción de alertas (admitido por el criterio). Tokens 6.728/583. |

## Los dos arreglos de contrato que salieron de acá (v4.1 → v4.3)

1. **Bebida (v4.2):** "solo se cotiza si el cliente la menciona; si no la menciona y la
   comida no incluye ninguna, va aviso en alertas en vez de sumarla."
2. **Sesiones partidas (v4.3):** "cada bloque continuo de simulador es una sesión propia;
   bloques separados por otra actividad se cotizan como sesiones independientes (1 h +
   merienda + 1 h = dos de 60, nunca una de 120); la combinación de sesiones aplica solo
   dentro de un bloque continuo." Además de la estabilidad, corrige el sentido comercial:
   partir el bloque no puede salir más barato que corrido.

## Salida cruda de la corrida final estable (R5, v4.3)

```json
{
  "precios_version": "2026-09-06.1",
  "evento": {"fecha": "2026-08-15", "dia_semana": "sábado", "categoria_dia": "fin_de_semana", "horario": "18:00-20:30", "personas_corren": 8, "personas_comen": 8},
  "modo_cotizacion": "carta",
  "paquete": {"nombre": "ninguno", "personas": 0, "precio_unitario": 0, "subtotal": 0},
  "simuladores": {"modalidad": "individual", "sesiones": [{"duracion_min": 60, "cantidad": 16, "precio_unitario": 29000, "subtotal": 464000}], "subtotal": 464000},
  "comida": {"items": [{"producto": "Box Cafe (Cafe + Tost)", "cantidad": 8, "precio_unitario": 11900, "subtotal": 95200}], "subtotal": 95200},
  "bebida": {"items": [], "subtotal": 0},
  "subtotal_lista": 559200,
  "descuentos": [{"concepto": "Evento fin de semana 10%", "porcentaje": 10, "monto": 55920}],
  "total": 503280,
  "total_redondeado": 503000,
  "sena_50": 251500,
  "estado": "completo",
  "datos_faltantes": [],
  "faltantes": [],
  "alertas": ["Merienda: supuesto Box Cafe (Cafe + Tost) por persona al no especificarse detalle.", "Los dos bloques de simulador separados por la merienda se cotizan como sesiones independientes de 60 min."]
}
```

## Conclusión

El test de reproducción hizo exactamente su trabajo: **encontró dos ambigüedades reales del
contrato** (una de negocio, una de cotización) y un bug de caché del frontend, forzó su
corrección, y con v4.3 el sistema reproduce **2/2** el mismo resultado bajo el criterio del
README. Es la misma lección de la Entrega 2, ahora con método: lo que no está escrito se
adivina distinto cada vez — hasta que se escribe.

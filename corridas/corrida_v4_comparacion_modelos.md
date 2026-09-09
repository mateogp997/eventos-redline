# Registro — comparación de modelos (mismo input, mismo criterio)

- **Fecha:** 2026-09-09, madrugada · Operador: Mateo García · frontend `index.html` → API Gemini
- **Entrada:** el hilo de Guido, idéntico al de las reproducciones R4/R5
  ([corrida_v4_guido_reproducciones.md](corrida_v4_guido_reproducciones.md))
- **Contrato:** v4.3 + precios `2026-09-06.1` · temperatura 0 en todos los casos
- **Criterio de éxito:** el de siempre (README) + coincidir con el resultado estable de
  referencia: carta, $503.280, sin bebida, 16 sesiones de 60 min

## Resultados

| Modelo | Total | ¿Pasa? | Detalle |
|---|---:|---|---|
| `gemini-flash-lite-latest` (más chico) | $291.200 | ❌ | Tomó el "$36.400 por persona" del hilo como precio acordado — pero esa oferta era para **otro cronograma** que el cliente descartó (Redline dijo "tengo que sacarte el costo" para el pedido final). 8 × $36.400. Le falta el criterio para atribuir la oferta a la variante correcta. Tokens 6.728/428. |
| `gemini-3.6-flash` (elegido) | $503.280 | ✅ | Reproduce el resultado de referencia 2/2 (R4 y R5). Tokens ~6.728/585. USD 0 en free tier; equivalente pago ~USD 0,007 por corrida. |
| `gemini-pro-latest` (más grande) | — | no evaluable | El nivel pro **no está incluido en el free tier** (cuota 0: "limit: 0, model: gemini-3.1-pro"). No hace falta subir: el flash ya pasa el criterio. |

## Decisión

Se opera con **`gemini-3.6-flash`**: es el modelo más chico disponible que cumple el
criterio de éxito. El escalón inferior (flash-lite) falla en el juicio más delicado del
contrato — a qué variante corresponde un precio acordado — que es exactamente el tipo de
error caro en un cotizador. La regla del curso aplicada con datos: se baja de modelo hasta
que falla, y se opera un escalón arriba de la falla.

Las tres corridas quedan conservadas (los dos resultados crudos relevantes están en este
registro y en el de reproducciones; la salida completa del flash-lite se transcribe abajo).

## Salida cruda del flash-lite (la que falla)

```json
{
  "modo_cotizacion": "acordado",
  "paquete": {"nombre": "acordado", "personas": 8, "precio_unitario": 36400, "subtotal": 291200},
  "total": 291200,
  "total_redondeado": 291000,
  "sena_50": 145500,
  "estado": "completo",
  "nota": "resumen; aritmética internamente consistente (8/8 checks) — el error es de atribución del precio, no de cuentas: por eso el criterio de éxito exige coincidir con la referencia y no solo pasar los checks"
}
```

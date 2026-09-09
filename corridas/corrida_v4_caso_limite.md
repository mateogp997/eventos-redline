# Registro de corrida — corrida_v4_caso_limite

- **Fecha de ejecución:** 2026-09-09T02:17:04 (hora local)
- **Herramienta:** frontend `index.html` del repo → API de Google Gemini (fetch directo) · Operador: Mateo García
- **Modelo:** `gemini-3.6-flash` · temperatura: 0 · max_tokens: 8192 · responseMimeType: application/json
- **Contrato:** `prompts/system_prompt.md` (v4.1) + `datos/precios.json` versión `2026-09-06.1`
- **Tokens:** entrada 6.260 · salida 410 → costo USD 0 en free tier (equivalente pago: USD 0,0063)
- **Estado:** incompleto · total: $0 · **objetivo: probar la conducta ante un hilo sin datos**

## Entrada (user prompt completo)

```
Cotizá este pedido de evento.

Fecha de hoy: 2026-09-08

Hilo de conversación (pegar tal cual, marcando quién habla en cada mensaje):
"""
Cliente: Hola! cuanto sale hacer un evento ahí?
"""

Datos que confirmé por otro canal (si hay):
- ninguno

Devolvé solo el JSON.
```

## Salida cruda del agente

```json
{
  "precios_version": "2026-09-06.1",
  "evento": {"fecha": "no indicada", "dia_semana": "no indicado", "categoria_dia": "semana", "horario": "no indicado", "personas_corren": 0, "personas_comen": 0},
  "modo_cotizacion": "carta",
  "paquete": {"nombre": "ninguno", "personas": 0, "precio_unitario": 0, "subtotal": 0},
  "simuladores": {"modalidad": "individual", "sesiones": [], "subtotal": 0},
  "comida": {"items": [], "subtotal": 0},
  "bebida": {"items": [], "subtotal": 0},
  "subtotal_lista": 0,
  "descuentos": [],
  "total": 0,
  "total_redondeado": 0,
  "sena_50": 0,
  "estado": "incompleto",
  "datos_faltantes": ["fecha exacta", "cantidad de personas", "tiempo de simulador", "si consumen comida o bebida"],
  "faltantes": [],
  "alertas": ["Referencia: sesión 60 min $29.000 por simulador. Eventos desde 6 personas tienen 10% desc. en fin de semana y 30% en día de semana (sobre carta)."]
}
```

## Verificación

- 8/8 checks automáticos en verde.
- **Conducta esperada ante entrada fuera del dominio, cumplida:** montos en cero, estado
  `incompleto`, los cuatro datos a preguntar listados y una única alerta con un precio de
  referencia útil para responderle al cliente. **Nada inventado** — la regla "no inventes
  datos que el cliente no dio" (contrato §4), probada en ejecución.

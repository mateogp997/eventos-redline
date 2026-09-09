# Registro de corrida — corrida_v4_selene

- **Fecha de ejecución:** 2026-09-09T02:15:38 (hora local)
- **Herramienta:** frontend `index.html` del repo → API de Google Gemini (fetch directo) · Operador: Mateo García
- **Modelo:** `gemini-3.6-flash` · temperatura: 0 · max_tokens: 8192 · responseMimeType: application/json
- **Contrato:** `prompts/system_prompt.md` (v4.1) + `datos/precios.json` versión `2026-09-06.1`
- **Tokens:** entrada 6.618 · salida 484 → costo USD 0 en free tier (equivalente pago: USD 0,0068)
- **Estado:** incompleto (falta horario) · modo: paquete (pro_bday) · total: $525.000 · seña: $262.500
- **Caso:** hilo real de Selene (iteraciones 1-2); fecha de hoy histórica 2026-08-15

## Entrada (user prompt completo)

```
Cotizá este pedido de evento.

Fecha de hoy: 2026-08-15

Hilo de conversación (pegar tal cual, marcando quién habla en cada mensaje):
"""
Cliente: El tema es que son 15 varones. Se puede hacer algo con tantos varones?
Redline: Te paso la propuesta: Pro Bday – Edición Tarde. Duración 2 horas. Incluye uso de los simuladores (si el grupo supera los 8 chicos, se organizan en dos equipos para que todos puedan disfrutar). Una merienda por persona a elección entre tostado de jamón y queso, chipa, medialunas, wrap, budín, café o té. Espacio para la torta. Valores: lunes a jueves $35.000 por persona; viernes a domingo y feriados $40.000 por persona. Mínimo 6 invitados, hasta 16 chicos.
Redline: Cronograma (2 horas): 0:00 a 0:30 Grupo A (8 chicos) en simuladores, Grupo B (7) merienda. 0:30 a 1:00 se intercambian. 1:00 a 1:30 todos juntos torta. 1:30 a 2:00 mini torneo final.
Cliente: La propuesta me encanta! Después te confirmo.
Cliente: Aún no definió si quiere que vengan las nenas, ahí se complica porque son 30.
Redline: Si quieren ahí podemos hacer un evento cerrado para ustedes, cerraríamos el local. Serían 3 horas y los simus para ustedes, con rotación rápida para que todos jueguen.
Cliente: Puede ser a la salida del cole, por ahí un martes? 20/10.
Redline: Te paso una propuesta tanto para 30 como para 10, así evalúan.
"""

Datos que confirmé por otro canal (si hay):
- ninguno

Devolvé solo el JSON.
```

## Salida cruda del agente

```json
{
  "precios_version": "2026-09-06.1",
  "evento": {"fecha": "2026-10-20", "dia_semana": "martes", "categoria_dia": "semana", "horario": "no indicado", "personas_corren": 15, "personas_comen": 15},
  "modo_cotizacion": "paquete",
  "paquete": {"nombre": "pro_bday", "personas": 15, "precio_unitario": 35000, "subtotal": 525000},
  "simuladores": {"modalidad": "por_turnos", "sesiones": [], "subtotal": 0},
  "comida": {"items": [], "subtotal": 0},
  "bebida": {"items": [], "subtotal": 0},
  "subtotal_lista": 525000,
  "descuentos": [],
  "total": 525000,
  "total_redondeado": 525000,
  "sena_50": 262500,
  "estado": "incompleto",
  "datos_faltantes": ["horario exacto"],
  "faltantes": [],
  "alertas": ["Cotización principal basada en Pro Bday para 15 chicos ($35.000 c/u en día de semana).", "Variante 10 chicos (Pro Bday): $350.000 total (seña $175.000).", "Variante 30 personas: supera el tope de Pro Bday (16 pers.) y requiere cierre exclusivo del local, cuyo precio no está definido."]
}
```

## Verificación

- 8/8 checks automáticos del frontend en verde.
- **Objetivo de la iteración 3 cumplido:** la cotización principal es la variante que la
  clienta aprobó (Pro Bday de 15, "La propuesta me encanta"), a precio de día de semana
  (martes 20/10 ✓); las variantes de 10 y 30 van en alertas, y el cierre exclusivo del
  local queda **sin precio inventado** (extra `null` en precios.json). Con v3, este caso
  cotizaba como principal la variante de 30 por carta.

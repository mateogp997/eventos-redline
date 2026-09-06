# Registro de corrida — corrida_v4_guido

- **Fecha de ejecución:** 2026-09-06T22:08:41 (hora local)
- **Herramienta:** frontend `index.html` del repo → API de Google Gemini (fetch directo)
- **Operador:** Mateo García
- **Modelo:** `gemini-3.6-flash` · temperatura: 0 · max_tokens: 8192 · responseMimeType: application/json
- **Contrato:** `prompts/system_prompt.md` (v4.1) + `datos/precios.json` versión `2026-09-06.1`
- **Tokens:** entrada 6.473 · salida 610 → costo USD 0 en free tier (equivalente pago: USD 0,0071, tarifas del 6/9/2026)
- **Estado:** completo · modo: carta · total exacto: $503.280 · total comunicado: $503.000 · seña: $251.500
- **Caso:** el mismo hilo real de Guido usado en las iteraciones 1-2 (fecha de hoy histórica 2026-08-10, para comparabilidad con las corridas v2/v3)

## Entrada (user prompt completo)

```
Cotizá este pedido de evento.

Fecha de hoy: 2026-08-10

Hilo de conversación (pegar tal cual, marcando quién habla en cada mensaje):
"""
Redline: Lo que nosotros tenemos programado es 1 hs de simulador, 30 min de merienda, y cerrar con la torta (serían los otros 30 min que restan). Si queres podemos armar una propuesta a tu medida, comentame si día de semana o finde, cantidad de niños, y supongo que será merienda.
Cliente: Y se puede armar 1 hora simulador, cortar 30 min para merienda y torta, y luego 30 min más de simulador?
Redline: Estuve viendo con mi jefe lo de cortar 30 min en el medio, se puede hacer pero tiene un recargo extra, quedaría en $36400 cada persona.
Cliente: Y hacer así? 1 hora simu, 30 merienda, 1 hora simu
Redline: Si, tengo que sacarte el costo. Y decime si ustedes traen la torta o queres que te demos nosotros acá (es aparte el costo, para 10 porciones está $70.000).
Cliente: Sería sábado o domingo. Hoy te confirmo a la tarde.
Cliente: Sabado 15. A las 18 hs. Sin torta, no quiere, solo merienda.
"""

Datos que confirmé por otro canal (si hay):
- Son 8 niños.

Devolvé solo el JSON.
```

## Salida cruda del agente

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
  "alertas": ["Merienda: supuesto Box Cafe (Cafe + Tost) por persona por falta de especificación.", "El cronograma personalizado (1h sim + 30m merienda + 1h sim) se cotiza como dos sesiones de 60 min por persona.", "Sin torta por indicación explícita del cliente."]
}
```

## Verificación automática del frontend

- ✅ Campos obligatorios presentes
- ✅ subtotal_lista = sims + comida + bebida
- ✅ Subtotales de items suman por rubro
- ✅ total = subtotal_lista − descuentos
- ✅ total_redondeado = total bajado al millar
- ✅ Seña = 50% del total redondeado
- ✅ Máximo 4 alertas
- ✅ precios_version coincide con datos/precios.json

## Revisión humana

- [x] Total y alertas revisados contra el hilo — con **observación de calibración**:
  el agente cotizó por **carta** ($503.280: 16 sesiones de 60 + 8 Box Cafe − 10%) porque el
  cronograma pedido (1h + merienda + 1h) tiene 30 min más de simulador que el paquete
  Merienda y la regla de precedencia v4.1 exige que el pedido "encaje" para ir a paquete.
  La v3 lo cotizaba como paquete ($58.000 × 8 = $464.000) con el extra en `faltantes`.
  Ambas lecturas son defendibles; queda planteado el ajuste de contrato "paquete +
  agregados menores" — decisión comercial pendiente del dueño (ver DECISIONES).
- [x] Además se generó la propuesta en PDF ("Propuesta Redline - Cumple de Guido.pdf")
  con esta cotización — primera corrida de punta a punta: hilo → JSON → PDF de marca.
- [ ] Respuesta enviada al cliente por: (caso histórico, no se envía)

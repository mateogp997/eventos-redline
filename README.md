# Entrega 2 — Cotizador de eventos de Redline Racing

**Materia:** MBA UCEMA
**Alumno:** Mateo Joaquín García Placeres
**Tarea recurrente elegida:** cotizar presupuestos de eventos privados (cumpleaños, corporativos, grupos) en Redline Racing, un bar de sim racing en Canning.

## La tarea

Todas las semanas llegan por WhatsApp e Instagram pedidos de presupuesto para eventos. Cada uno depende de una combinación de variables: día de la semana (fin de semana cuesta más porque ya se llena solo), cantidad de personas que corren y cantidad que comen (no siempre coinciden), tiempo de simulador por persona o tiempo total con turnos, tipo de comida (merienda, hamburguesa simple o doble), bebida con o sin alcohol. Hoy eso lo resuelve de cabeza quien atiende el WhatsApp, y el resultado varía según quién y cuándo.

El objetivo del contrato es que a partir del hilo de WhatsApp tal cual llegó, el agente devuelva un presupuesto desglosado en JSON, con los supuestos declarados y lo que falta preguntar, para que el equipo lo revise y lo comunique al cliente.

## Estructura del repo

```
README.md
prompts/
  system_prompt_v1.md        ← contrato inicial
  system_prompt_v2.md        ← después de la iteración 1
  system_prompt_v3_final.md  ← después de la iteración 2 (versión final)
  user_prompt_v1.md          ← plantilla inicial
  user_prompt_v2.md          ← plantilla después de la iteración 1 (final)
corridas/
  corrida_1_cumple13_*       ← caso usado para diagnosticar la iteración 1
  corrida_2_guido_*          ← corrida final 1 (sábado, 8 chicos, merienda)
  corrida_3_selene_*         ← corrida final 2 (martes, 15 o 30 chicos, turnos)
  corrida_4_alejandro_*      ← corrida final 3 (domingo, 6 adultos, cena)
```

Cada corrida tiene su `_input.md` (el user prompt exacto que se mandó) y uno o más `_output_vN.json` (la respuesta tal cual la devolvió el modelo con la versión N del system prompt). Los tres casos finales son hilos reales de WhatsApp de agosto 2026, con los datos del cliente recortados a lo necesario.

## Cómo se corre

1. Crear un Proyecto en claude.ai y pegar `prompts/system_prompt_v3_final.md` en las instrucciones del proyecto.
2. Abrir un chat nuevo dentro del proyecto.
3. Copiar `prompts/user_prompt_v2.md`, completar la fecha de hoy y pegar el hilo de WhatsApp marcando cada mensaje con `Cliente:` o `Redline:`.
4. La respuesta es un JSON. No se manda al cliente: se lee el total y las alertas, y se le escribe al cliente aparte.

## Las seis piezas

Todas viven en el system prompt, que es la parte fija. El user prompt solo lleva lo que cambia en cada corrida: la fecha de hoy y el hilo del cliente.

| Pieza | Dónde está | Qué dice |
|---|---|---|
| 1 · Rol | System | Encargado de eventos de Redline, con años cotizando |
| 2 · Contexto | System | 8 simuladores, horarios, carta completa con precios, paquetes de evento, condiciones (mínimo 6, descuentos, seña 50%) |
| 3 · Tarea | System | Calcular el presupuesto del pedido del user prompt y devolverlo en el JSON indicado |
| 4 · Restricciones | System | Usar solo precios del documento; no inventar datos; supuestos por defecto declarados; capacidad; distinguir quién corre de quién come; máximo 4 alertas; nada fuera del JSON |
| 5 · Formato | System | JSON con evento, modo de cotización, paquete, simuladores, comida, bebida, descuentos, total, seña, estado, faltantes y alertas |
| 6 · Ejemplos | System | Cuatro pares entrada→salida: dos por carta, uno incompleto, uno por paquete |
| Datos de hoy | User | Fecha de hoy, hilo de WhatsApp con voces marcadas, datos confirmados por otro canal |

## Iteración 1 — el modelo no sabía quién hablaba

**Antes (v1).** Corrida 1, un cumpleaños de 8 chicos de 13 años. El user prompt pedía "pegar el mensaje del cliente tal cual", así que pegué el hilo completo, que mezclaba mensajes del cliente y respuestas nuestras. Resultado (`corrida_1_cumple13_output_v1.json`):

- Trató todo el hilo como voz del cliente. Alertó que "el combo con cargo adicional no existe en la carta", cuando ese combo lo había ofrecido Redline.
- Escribió productos que no existen con ese nombre: `Papas Clásicas` con tilde, `Gaseosas` en genérico. Para un output que se supone que es dato, eso lo vuelve inusable.
- Nueve alertas, varias repitiendo reglas del sistema ("esto es cotización, no reserva", "son menores, no se sirve alcohol" cuando nadie pidió alcohol).

**Qué toqué.**

- *User prompt:* el hilo se pega con prefijos `Cliente:` / `Redline:`.
- *Tarea:* aclarar que lo dicho por Redline en el hilo vale como parte del acuerdo.
- *Formato:* nombres de producto exactamente como figuran en la carta, sin tildes, con ejemplos; agregué `Gaseosa` como ítem genérico de eventos a la carta.
- *Restricciones:* máximo 4 alertas, y solo para supuestos que cambian el precio, fecha/horario o capacidad.

**Después (v2).** Lo corrí dos veces para separar efectos:

- Con el hilo sin marcar voces (`_output_v2_sin_voces.json`): total idéntico, nombres exactos, cuatro alertas útiles. El formato y el tope de alertas funcionaron.
- Con el hilo marcado (`_output_v2.json`): el cambio es sutil pero real. Ahora dice "2 porciones con cheddar **ofrecidas por Redline**", y en la última alerta detecta que nuestra oferta describía un paquete con una bebida por persona, lo que contradecía el supuesto por defecto de dos. Ese aviso fue la pista de la iteración 2.

## Iteración 2 — el contrato cotizaba distinto a como cotiza Redline

**Antes (v2).** Corrí los tres casos reales. Formato perfecto, cuentas correctas, pero los totales no coincidían con lo que Agustina (quien atiende eventos) había ofrecido en esos mismos hilos:

| Caso | Contrato v2 (carta − descuento) | Ofrecido por Redline |
|---|---|---|
| Guido, sábado, 8 chicos | $62.910 por persona | ~$58.000 |
| Selene, martes, 15 chicos | $26.250 por persona | $35.000 |
| Alejandro, domingo, 6 adultos | $44.760 por persona | $53.100 |

El contrato decía "no hay menú de evento cerrado, se cotiza por carta con descuento". Pero en la práctica Redline vende paquetes por persona (Merienda, Cena, Pro Bday) con precio fijo según día. Las dos reglas de precio convivían sin que nadie lo hubiera escrito. El modelo no lo inventó: en dos de tres casos avisó que "el precio ofrecido no coincide con el de este documento".

Además, corriendo dos veces el mismo input con la misma v2 (`_output_v2_a.json` y `_output_v2_b.json`), el resultado cambió: en Guido cotizó 1h + 1h como dos sesiones de 60 en la primera y como una de 120 en la segunda; en Alejandro usó 1 bebida por persona en la primera y 2 en la segunda. Lo que no está en el contrato, el modelo lo adivina, y adivina distinto cada vez.

**Qué toqué.**

- *Contexto:* tabla de paquetes (Merienda $51.000/$58.000, Cena $55.000/$63.000, Pro Bday $35.000/$40.000, semana/finde), extras (torta, recargos por cambio de cronograma) y regla de precedencia: si el pedido encaja en un paquete o Redline ya lo ofreció, va paquete; la carta con descuento es solo para lo que no encaja. Nunca mezclar.
- *Formato:* campo `modo_cotizacion` (`paquete` / `carta`) y bloque `paquete`.
- *Ejemplos:* uno nuevo por paquete, y a los de carta les agregué por qué no encajan en paquete.

Los precios de paquete los saqué de los mismos hilos de WhatsApp. Había variación (merienda a $56.000 y $58.000 en dos chats de la misma semana): tomé el más alto como precio de lista y definí los de semana.

**Después (v3).**

- Guido: paquete Merienda a $58.000, igual que lo ofrecido. El cronograma pedido tiene 30 min más de simulador que el paquete: quedó en `faltantes` como recargo sin precio, no inventado.
- Selene: 30 chicos no entran en Pro Bday (tope 16) → carta, con el cierre del local como faltante sin precio; la variante de 15 quedó en alertas a $35.000 × 15. Cotizó como principal la variante de 30, cuando el cliente había aprobado la de 15; es defendible pero no era lo que esperaba.
- Alejandro: fue a carta ($268.560) porque lo ofrecido ($53.100 por persona, 1 h de sim + cena, sin la segunda media hora) no es exactamente un paquete. Lo marcó en la primera alerta. Acá queda un agujero: el contrato no dice qué hacer cuando Redline ya dio un precio a medida en el hilo.

## Las tres corridas con el prompt final

| | Guido | Selene | Alejandro |
|---|---|---|---|
| Input | `corrida_2_guido_input.md` | `corrida_3_selene_input.md` | `corrida_4_alejandro_input.md` |
| Output | `corrida_2_guido_output_v3.json` | `corrida_3_selene_output_v3.json` | `corrida_4_alejandro_output_v3.json` |
| Día | sábado | martes | domingo |
| Personas | 8 chicos | 30 (o 15) | 6 adultos |
| Modo | paquete | carta | carta |
| Total | $464.000 | $686.700 (parcial) | $268.560 |
| Estado | incompleto | incompleto | incompleto |

Las tres respetan el mismo esquema JSON, nombres de producto exactos, máximo 4 alertas, cuentas verificadas a mano (subtotales, descuento, seña redondeada a la centena). El JSON de Alejandro quedó cortado al copiarlo desde el chat; se conserva tal cual con una nota.

## Reflexión — qué aprendí del contrato

Empecé pensando que esto era una forma incómoda de hacer algo que preferiría programar: un formulario que calcule el presupuesto. Después de las corridas entendí por qué no es lo mismo. El formulario necesita que alguien lea el WhatsApp y lo traduzca a campos. El contrato lee el hilo como llegó, con "somos como 15, algunos corren y otros solo comen", y devuelve los campos ya cargados más la lista de lo que hay que preguntar. La parte que vale es la extracción, no el cálculo.

Lo segundo, y lo que más me sirvió: **el contrato me mostró que mi lista de precios y mi vendedora no dicen lo mismo.** Yo escribí el system prompt convencido de que cotizábamos por carta con descuento. Los hilos reales mostraron que Agustina cotiza paquetes por persona, y que esos paquetes ni siquiera tienen un precio único. No lo descubrí leyendo los chats; lo descubrí porque el modelo, al no tener los paquetes en el contrato, cotizó otra cosa y avisó que no coincidía. Un empleado nuevo con las mismas instrucciones habría hecho lo mismo, y probablemente sin avisar.

Lo tercero: lo que no está escrito se adivina, y se adivina distinto cada vez. Mismo prompt, mismo input, dos resultados. Cada vez que el output me sorprendió, la pregunta "¿cuál de las seis piezas falta?" tuvo respuesta concreta: una vez el user prompt, otra el formato, otra el contexto. El diagnóstico funciona.

Lo que queda pendiente para una iteración 3: una regla para cuando Redline ya dio un precio a medida en el hilo (hoy el contrato cotiza por carta y puede salir más barato que lo prometido), un precio para el corte de 30 minutos en el medio del bloque de simulador y para el cierre exclusivo del local, y una decisión sobre qué variante cotizar como principal cuando el cliente está entre dos escenarios.

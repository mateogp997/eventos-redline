# USER PROMPT — plantilla por corrida (vigente; sin cambios desde la v2)

Cotizá este pedido de evento.

Fecha de hoy: [YYYY-MM-DD]

Hilo de conversación (pegar tal cual, marcando quién habla en cada mensaje):
"""
Cliente: [mensaje del cliente]
Redline: [nuestra respuesta, si la hubo]
Cliente: [mensaje del cliente]
"""

Datos que confirmé por otro canal (si hay):
- [ej: por audio confirmó que son 12 y no 10]

Devolvé solo el JSON.

---

Nota: el frontend (`index.html`) arma este user prompt automáticamente con la fecha del día
y el hilo pegado en el formulario. Esta plantilla queda para correr el contrato a mano
(por ejemplo en un Proyecto de claude.ai).

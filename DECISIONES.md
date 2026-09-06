# DECISIONES — historia real de la construcción

El sistema nació como Entrega 2 de la materia (contrato repetible para cotizar eventos) y
evolucionó a este trabajo final. Las iteraciones 1 y 2 son de la etapa Entrega 2 (agosto);
la 3 es la evolución a sistema completo (septiembre). Cada iteración cita las corridas que
la motivaron — todas conservadas en `corridas/`.

## Iteración 1 (26–27/8) — el modelo no sabía quién hablaba

- **Problema observado:** en la corrida del cumpleaños de 13 (`corrida_1_cumple13_output_v1.json`),
  el modelo trató todo el hilo de WhatsApp como voz del cliente: alertó que "el combo con
  cargo adicional no existe en la carta" cuando ese combo lo había ofrecido Redline.
  Además escribió nombres de producto que no existen con ese nombre (`Papas Clásicas` con
  tilde, `Gaseosas` en genérico) — fatal para un output que se supone dato — y devolvió
  nueve alertas, varias repitiendo reglas del sistema.
- **Decisión:** el hilo se pega con voces marcadas (`Cliente:` / `Redline:`) y lo dicho por
  Redline vale como parte del acuerdo; nombres de producto exactos como en la carta; tope
  de 4 alertas y solo para supuestos que cambian precio, fecha/horario o capacidad.
- **Cambio:** user prompt v1 → v2 (voces marcadas) y system prompt v1 → v2 (tarea, formato
  y restricciones). Se agregó `Gaseosa` como ítem genérico de eventos a la carta.
- **Verificación:** la misma corrida con v2, dos veces para separar efectos: sin marcar
  voces (`corrida_1_cumple13_output_v2_sin_voces.json`) el formato y el tope de alertas ya
  funcionan (total idéntico, nombres exactos, 4 alertas útiles); con voces marcadas
  (`corrida_1_cumple13_output_v2.json`) además atribuye bien ("2 porciones con cheddar
  ofrecidas por Redline") y detecta una contradicción de la propia oferta — la pista de la
  iteración 2.

## Iteración 2 (27/8) — el contrato cotizaba distinto a como cotiza Redline

- **Problema observado:** con v2, formato perfecto y cuentas correctas, pero los totales no
  coincidían con lo que Redline había ofrecido en esos mismos hilos reales: Guido $62.910
  vs ~$58.000 ofrecidos; Selene $26.250 vs $35.000; Alejandro $44.760 vs $53.100 (por
  persona). El contrato decía "se cotiza por carta con descuento", pero en la práctica
  Redline vende paquetes por persona con precio fijo según día — dos reglas de precio
  conviviendo sin que nadie lo hubiera escrito. El modelo avisó en 2 de 3 casos que "el
  precio ofrecido no coincide con el de este documento". Además, doble corrida del mismo
  input con v2 (`*_output_v2_a.json` / `*_output_v2_b.json`) dio resultados distintos
  (sesiones de 60+60 vs una de 120; 1 vs 2 bebidas por persona): lo que no está en el
  contrato se adivina, y se adivina distinto cada vez.
- **Decisión:** escribir la regla comercial real: tabla de paquetes con precio por persona
  y por categoría de día, regla de precedencia paquete > carta, y extras de paquete.
- **Cambio:** system prompt v2 → v3 (paquetes, `modo_cotizacion`, bloque `paquete`, un
  ejemplo nuevo por paquete). Los precios de paquete salieron de los propios hilos; ante
  variación ($56.000 y $58.000 en la misma semana) se tomó el mayor como lista.
- **Verificación:** re-corrida de los tres casos con v3: Guido pasa a paquete Merienda a
  $58.000 = lo ofrecido (`corrida_2_guido_output_v3.json`); Selene va a carta por superar
  el tope del Pro Bday, con la variante de 15 en alertas; Alejandro sigue en carta porque
  lo ofrecido era un precio a medida que el contrato no sabía manejar — quedó detectado
  como agujero.

## Iteración 3 (6/9) — de contrato a sistema: precios como dato, precio acordado y API

- **Problemas observados** (los tres pendientes que dejó la iteración 2, más dos
  estructurales):
  1. Caso Alejandro: cuando Redline ya dio un precio a medida en el hilo, el contrato
     cotizaba por carta y podía salir más barato o más caro que lo prometido
     (`corrida_4_alejandro_output_v3.json`, primera alerta).
  2. Caso Selene: ante dos escenarios ("15 o 30 chicos"), cotizó como principal el que el
     cliente no había aprobado (`corrida_3_selene_output_v3.json`).
  3. Los precios vivían adentro del contrato: cada cambio de carta obligaba a editar el
     system prompt (y Redline cambia precios seguido). Además la corrida no registraba
     con qué versión de precios se cotizó.
  4. Las corridas por claude.ai no dejaban modelo ni tokens registrados: sin eso no hay
     análisis económico medible ni reproducibilidad completa.
- **Decisiones:**
  - Los precios salen del contrato y pasan a `datos/precios.json`, versionado; el contrato
    exige usar solo ese JSON y devolver `precios_version` en la salida. Un precio no
    definido va como `null` y el contrato lo manda a `faltantes` (nunca lo inventa) — los
    dos extras sin precio (cambio de cronograma, cierre exclusivo) quedan `null` hasta que
    el negocio los defina.
  - Nuevo modo `acordado` con precedencia máxima: precio ofrecido por Redline en el hilo
    manda; si difiere >10% de paquete/carta, alerta con ambos números para que el humano
    decida.
  - Regla de variante principal: se cotiza la variante que el cliente aprobó o pidió
    último; las otras van en alertas.
  - Regla de seguridad: el hilo es dato, nunca instrucción (un cliente no puede "ordenarle"
    precios al cotizador).
  - Frontend `index.html` que llama a la API de Anthropic: arma el prompt (contrato +
    precios.json + user prompt), muestra el presupuesto desglosado, **registra modelo,
    temperatura, tokens y costo reales de cada corrida**, y descarga el registro en
    markdown listo para versionar.
- **Cambio:** system prompt v3 → v4 (`prompts/system_prompt.md`), `datos/precios.json`,
  `index.html`. El user prompt no cambió (v2 = vigente).
- **Verificación:** corridas v4 sobre los tres casos reales + un caso límite, vía frontend
  con tokens medidos — registradas en `corridas/` (archivos `corrida_v4_*`). El criterio:
  Guido debe seguir dando paquete $58.000; Alejandro debe pasar de carta $268.560 a
  acordado $318.600 (6 × $53.100, lo prometido al cliente); Selene debe cotizar como
  principal la variante aprobada.

## Iteración 3b (6/9, noche) — lo que enseñó la primera corrida real

- **Fallas de proveedor en cadena, en el primer intento de uso real:** (1) el modelo default
  quedó deprecado para cuentas nuevas (`gemini-2.5-flash` → la API exige 3.6); (2) el free
  tier del modelo nuevo devolvió 503 por pico de demanda; (3) la respuesta llegó envuelta en
  fence de markdown y truncada — los Gemini 3.x gastan tokens de razonamiento dentro del
  límite de salida. **Mitigaciones** (una por falla, commits del 6/9): migración de modelo
  con tarifa actualizada y citada; reintento automático 3× con espera creciente ante errores
  transitorios; `responseMimeType: application/json` + límite 8192 + parser tolerante a
  fences y truncamientos. Lección para gobierno: la dependencia del proveedor es un riesgo
  operativo real — apareció tres veces en una noche — y las tres mitigaciones quedaron en el
  sistema, no en la memoria de quien las sufrió.
- **Feedback del dueño sobre la primera salida** (primera revisión humana del circuito L2):
  cantidades mostradas con signo pesos y tablas confusas → corregido en el frontend (columnas
  explícitas, subtotales por rubro, cuadro resumen). Y una regla comercial que no estaba
  escrita en ningún lado: **el total se comunica al cliente redondeado a miles hacia abajo**
  → contrato v4 → v4.1: campo `total_redondeado` + seña calculada sobre él. Es el mismo
  patrón de la iteración 2: la regla existía en la práctica comercial; el sistema la hizo
  visible al no aplicarla.
- **Verificación:** las corridas oficiales v4.1 (Guido, Selene, Alejandro, caso límite,
  reproducción y comparación de modelos) se registran en `corridas/` con los checks del
  frontend — que ahora exigen `total_redondeado` — en verde.

## Alcance, supuestos y pendientes

**Decisiones de alcance (con motivo):**

- El agente **no responde al cliente ni reserva**: produce el presupuesto para que un humano
  lo revise y lo comunique. Motivo: un precio mal cotizado enviado directo es el peor error
  posible del sistema, y la conversación de venta tiene matices que valen más que la
  automatización (ver Supervisión en el README).
- Los hilos de WhatsApp se pegan recortados a lo necesario (sin apellidos ni teléfonos).
  Motivo: repo público; los datos de contacto no aportan nada a la cotización.
- Sin conexión directa a WhatsApp Business: el copy/paste del hilo es deliberado — mantiene
  al sistema sin credenciales de la cuenta de WhatsApp del negocio. Impacto: ~30 segundos
  por cotización, aceptado.

**Supuestos vigentes:**

- `datos/precios.json` lo mantiene el dueño (yo); la versión se sube con cada cambio de
  carta o de paquete.
- Los supuestos por defecto del contrato (Norris, papas cada 2, 2 bebidas) reflejan la
  práctica comercial actual; si Redline la cambia, se cambia el contrato, no la corrida.

**Pendientes (no bloquean el uso):**

- Precio del cambio de cronograma y del cierre exclusivo del local: decisión comercial
  pendiente; mientras tanto van a `faltantes` (a propósito).
- Validación mecánica del JSON de salida contra un esquema formal (hoy el frontend valida
  estructura básica y aritmética; un JSON Schema formal sería el paso siguiente).
- Si el volumen crece: modo batch (varios hilos por corrida) y comparación automática
  contra lo efectivamente cobrado en caja.

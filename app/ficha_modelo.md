# Ficha del modelo · Riesgo de mala reseña (Olist)

**Equipo:** Ruiz Melissa, Rodrigo y Cesar · **Tipo de modelo:** XGBoost (un conjunto de árboles de decisión que aprende de pedidos pasados). Archivos técnicos: `modelo_riesgo.joblib` y `preparacion.py`.

## 1. Para qué sirve (y para qué no)
- **Sirve para:** ordenar los pedidos **recién entregados** según la probabilidad de que el cliente deje una reseña de 1 o 2 estrellas, para que atención al cliente llame primero al **10% más riesgoso** (su capacidad diaria) y resuelva el problema antes de la reseña.
- **No debe usarse para:** castigar, sancionar o calificar vendedores o transportistas; decidir reembolsos o beneficios de forma automática; predecir antes de la entrega (necesita la fecha real de entrega); ni para pedidos cancelados o no entregados.

## 2. Datos
- **Origen:** base de datos pública de Olist (esquema `final` del curso), pedidos reales de Brasil.
- **Periodo:** compras del 2016-09-15 al 2018-05-31.
- **Cuántos:** 77,304 pedidos entregados y con reseña, de 80,450 pedidos en la tabla de pedidos. Tasa de mala reseña: 13.5%.
- **Qué se excluyó y por qué:** 2,581 pedidos no entregados (cancelados, en camino, etc.) y 2 sin fecha de entrega registrada, porque el modelo predice *en el momento de la entrega*; y 563 pedidos entregados sin reseña, porque no sabemos cómo terminaron. Tampoco se usa nada de la reseña (texto, fechas, puntaje) como variable: esa información no existe cuando el modelo hace la predicción.

## 3. Qué tan bien funciona
Validación honesta por fecha: entrenamos con pedidos anteriores al 2018-03-01 y probamos con 20,422 pedidos posteriores que el modelo no había visto.
- **De cada 10 clientes que llamamos, unos 6.7 iban a dejar una mala reseña** (tasa de acierto al llamar al 10% más riesgoso: 0.668).
- Si se llamara **al azar**, solo unos 1.5 de cada 10 (0.146).
- Si se llamara **a los más retrasados** (regla simple), unos 6.2 de cada 10 (0.621).
- En un periodo con **pocos retrasos** (abril–mayo 2018) el modelo bajó a 4.5 de cada 10 (0.450), mientras que la regla bajó a 3.4 (0.339).

## 4. Límites conocidos
- Depende mucho del **retraso**: si la logística mejora o cambia la forma de calcular la fecha prometida, la precisión baja (ya lo vimos en abril–mayo).
- No ve el **producto** en sí (calidad, si coincide con el anuncio) ni el trato del vendedor: muchas malas reseñas por "producto distinto" o "defectuoso" no son detectables con estos datos.
- Se entrenó con 2016–mayo 2018; patrones nuevos (huelgas, nuevas regiones o categorías, Black Friday) pueden sorprenderlo.
- Es una **probabilidad**, no una certeza: en validación detecta 46% de las malas reseñas al llamar solo al 10%.

## 5. A quién afecta un error
- **Falso positivo** (llamamos a un cliente que estaba satisfecho, 677 casos en validación): costo de una llamada y una pequeña molestia para el cliente. Es un error barato.
- **Falso negativo** (no llamamos a un cliente que sí deja 1–2 estrellas, 1,624 casos en validación): se pierde la oportunidad de resolver el problema; afecta la reputación del vendedor y de Olist.
- **Riesgo de mal uso:** como el retraso pesa tanto (probabilidad media de 0.57 en pedidos retrasados vs 0.09 a tiempo), usar el puntaje para **sancionar vendedores** sería injusto: muchos retrasos son del transportista o de la región del cliente, no del vendedor. Tampoco debe usarse para dar un trato peor a clientes de regiones lejanas.

## 6. Recomendación
- Usarlo como **lista de prioridad diaria**: ordenar los pedidos entregados del día por probabilidad y llamar al 10% de arriba (lo que alcanza la capacidad del equipo). La app marca "riesgo alto" cuando la probabilidad supera 26%, el corte del 10% más riesgoso en los datos históricos; como la cantidad de retrasos cambia de un mes a otro, ese corte puede marcar más o menos del 10% en un día dado, así que manda el orden, no el corte. La persona de atención decide qué ofrecer; el modelo no toma decisiones sobre el cliente.
- **Supervisión humana:** revisar cada semana una muestra de llamadas para ver si el problema era real, y registrar el resultado para reentrenar.
- **Revisión:** medir la precisión en el 10% **cada mes** con las reseñas que llegaron; reentrenar y recalcular el umbral cada trimestre, o antes si la tasa de retraso cambia mucho o la precisión cae por debajo de la regla del retraso.

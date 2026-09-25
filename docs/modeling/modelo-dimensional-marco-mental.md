# Modelo dimensional: marco mental para distinguir Dimensions y Facts

## La idea principal

Cuando exista una duda entre **Dimension** y **Fact**, empezar con tres preguntas:

> **Dimension = ¿qué es algo?**<br>
> **Fact = ¿qué pasó?**<br>
> **Grain = ¿qué representa exactamente una fila?**

Es una regla de orientación, no un sustituto del análisis del negocio. La clasificación final depende del significado funcional del dato y del nivel de detalle requerido.

[Abrir la chuleta visual: Fact Table vs Dimension Table vs Factless Fact](../../assets/images/fact-vs-dimension-vs-factless-fact.png)

## Dimension: «¿qué es algo?»

Una Dimension describe una entidad, clasificación o contexto. Por ejemplo:

- Airline: qué aerolínea es;
- Airport: qué aeropuerto es;
- Country: qué país es;
- Aircraft Type: qué tipo de avión es;
- Date: qué fecha es y qué propiedades de calendario tiene.

Sus atributos permiten describir, agrupar y filtrar la entidad: nombre, código, país, categoría, descripción o grupo.

Una Dimension no tiene que ser inmutable. Una aerolínea puede cambiar de nombre o alianza y seguir siendo una Dimension. Si es necesario conservar la verdad histórica, puede emplearse una técnica como **Slowly Changing Dimension Type 2**.

La regla no es «dato fijo para siempre», sino:

> **Una Dimension describe qué es algo en el contexto analítico.**

## Fact: «¿qué pasó?»

Una Fact representa un hecho, ocurrencia, transacción o evento:

- un movimiento de vuelo;
- una venta;
- una reserva;
- una llamada;
- un envío;
- una incidencia;
- una huelga registrada en una fecha.

Una Fact suele relacionarse con varias Dimensions para explicar quién, qué, dónde, cuándo y cómo se clasifica el hecho.

```text
                  dim_date
                     |
dim_airline ---> fact_flight <--- dim_airport
                     |
                dim_aircraft
```

Una Fact puede contener medidas, claves foráneas, identificadores de negocio y atributos técnicos necesarios. El número de columnas no determina por sí solo si el diseño es correcto.

## Grain: la primera decisión de una Fact

Antes de diseñar o revisar una Fact hay que poder completar esta frase:

> **Una fila representa...**

Ejemplos:

- una fila = un movimiento de vuelo;
- una fila = una línea de venta;
- una fila = una reserva;
- una fila = un evento;
- una fila = un evento por día.

El grain debe expresarse como una frase de negocio. Encontrar una combinación de columnas técnicamente única no basta.

> **Clave técnica ≠ grain funcional.**

Después de declarar el grain, cada columna debe validarse contra él. Una edad de pasajero no cabe en una Fact cuyo grain sea un vuelo, porque un vuelo puede incluir muchos pasajeros. Un tipo de aeronave sí puede estar definido al nivel del vuelo, aunque todavía haya que decidir si debe residir como atributo en una Dimension.

## Factless Fact

Una **Factless Fact** registra que algo ocurrió o que una relación existió, aunque no tenga una medida numérica.

```text
fecha       evento
2026-01-10  huelga
2026-02-03  tormenta
```

No necesita pasajeros, euros, kilos o duración para ser un hecho. Su valor reside en registrar la ocurrencia y relacionarla con sus Dimensions.

## Aplicación al Event Tracker

El Event Tracker contiene sucesos asociados a fechas, como huelgas, meteorología, grandes eventos, problemas IT, guerra, drones o ATC. Conceptualmente responde mejor a «qué pasó» que a «qué es algo», por lo que es un candidato a **Factless Fact**.

La decisión no debe cerrarse sin confirmar con Business el grain funcional. Una fila podría representar:

- un evento;
- un evento en un día concreto;
- una anotación contextual asociada a una fecha.

Conocer columnas como `datum`, `lfd_nummer`, `kategorie`, `radius` o `ereignis_effekt` no resuelve por sí solo qué significa una fila, si un evento puede abarcar varios días o si pueden existir varios eventos el mismo día.

## Secuencia de decisión

1. Preguntar si la tabla describe una entidad o contexto. Si es así, probablemente es una Dimension.
2. Preguntar si registra algo que ocurrió. Si es así, probablemente es una Fact.
3. Si es una Fact, declarar en una frase qué representa una fila.
4. Validar que todas las medidas y relaciones existan a ese mismo grain.
5. Consultar a Business cuando la semántica de una fila o de una clave no esté demostrada.

## Lecturas relacionadas

- [Revisión de una Fact ancha](dimensional-modeling-interview-case.md): proceso para clasificar columnas y rediseñar una Fact sin reducirla de forma arbitraria.
- [Star schema en Silver o Gold](star-schema-silver-vs-gold.md): dónde deben vivir el grain, las claves, la historia y las relaciones canónicas.
- [Snowflakes, outriggers y bridge tables](snowflakes-outriggers-and-bridge-tables.md): patrones para relaciones que no caben en una estrella simple.

# Normalization

La normalización es el proceso de organizar los datos de una base de datos para reducir la redundancia y mejorar la integridad de los datos. Se aplica a través de formas normales sucesivas, cada una con requisitos más estrictos.

---

## 1st Normal Form (1NF)

- Todos los atributos son atómicos (no divisibles)
- La tabla contiene una clave primaria única (PK)
- La PK no contiene atributos nulos
- No debe existir variación en el número de columnas
- Los campos no clave deben identificarse por la clave (Dependencia Funcional)
- Debe existir una independencia del orden tanto de las filas como de las columnas

---

## 2nd Normal Form (2NF)

- Una relación está en 2FN si está en 1FN
- Los atributos que no forman parte de ninguna clave dependen de forma completa de la PK (Reflexiva)
- Puedo obtener los datos de las otras columnas a través de su PK (Aumentativa)
- Podemos usar llaves foráneas (FK) para obtener datos de las otras tablas (Transitiva)
- Si tengo una tabla A → B y B → C, NO necesito A → C para poder obtener los datos

---

## 3rd Normal Form (3NF)

- Una relación está en 3FN si está en 2FN
- Ningún atributo no clave depende transitivamente de la PK (no hay dependencias entre columnas no clave)

---

## 4th Normal Form (4NF)

- No podemos repetir datos en una tabla
- Solo tenemos combinaciones únicas
- Todas las llaves van a poder ser, sí o sí, PKs

---

## Can a NoSQL database be normalized?

**Short answer:** technically yes, but in most cases you **shouldn't** — it defeats the reason you chose NoSQL in the first place.

- **Relational normalization** is designed to remove redundancy and update anomalies by splitting data across many tables linked by FKs, reassembled at query time with `JOIN`s.
- **Non-relational databases** favor a **denormalized** model where data that is read together is stored together — trading some redundancy for huge gains in read performance and horizontal scalability.

You *can* simulate normalization in NoSQL by referencing documents by ID (e.g. MongoDB `$lookup`, DynamoDB separate items), but:
- Most NoSQL engines do **not** enforce foreign keys or referential integrity — it becomes the application's responsibility.
- Cross-document transactions are limited or expensive.
- "Joins" in NoSQL are generally slower and harder to scale.

**Rule of thumb:**
- Highly structured data, integrity critical, complex queries → **relational + normalize**
- Scale, flexible schemas, document-shaped reads → **NoSQL + model around access patterns**

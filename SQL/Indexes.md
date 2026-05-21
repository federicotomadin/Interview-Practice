# Indexes in SQL

Un índice es una estructura de datos auxiliar que el motor de base de datos mantiene para acelerar la búsqueda de filas. Sin índice, cada query hace un **full table scan** (O(n)). Con un índice adecuado, la búsqueda puede bajar a **O(log n)** o incluso O(1).

## Purpose of Indexes

- **Improve Query Performance:** Aceleran el `WHERE`, `JOIN`, `ORDER BY` y `GROUP BY`.
- **Enforce Uniqueness:** Los índices únicos garantizan que no haya duplicados en la columna indexada.
- **Primary Key:** La PK es internamente un índice clustered que asegura unicidad y acceso rápido por fila.

---

## Clustered Index

Un **clustered index** determina el **orden físico de almacenamiento** de las filas en disco. Los datos de la tabla *son* el índice — las filas se guardan ordenadas según la columna(s) del índice.

- **Solo puede haber uno por tabla** (porque las filas solo pueden estar ordenadas de una manera física).
- Por default, la **Primary Key** crea automáticamente el clustered index en SQL Server.
- Las lecturas por rango (`BETWEEN`, `>`, `<`) son muy eficientes porque las filas están físicamente contiguas en disco.

```
Tabla Policies (clustered por PolicyId):

Páginas de datos:
[PolicyId=1 | HolderId=A | Status=Active | ...]
[PolicyId=2 | HolderId=B | Status=Expired| ...]
[PolicyId=3 | HolderId=A | Status=Active | ...]
...

SELECT * FROM Policies WHERE PolicyId BETWEEN 100 AND 200
→ Va directo al bloque físico donde están esas filas. Muy rápido.
```

```sql
-- SQL Server: la PK crea el clustered index automáticamente
CREATE TABLE Policies (
    PolicyId INT PRIMARY KEY,   -- ← clustered index implícito
    HolderId INT,
    Status NVARCHAR(20)
);

-- O explícito:
CREATE CLUSTERED INDEX IX_Policies_PolicyId ON Policies (PolicyId);
```

---

## Non-Clustered Index

Un **non-clustered index** es una estructura **separada** de los datos. Contiene las columnas indexadas + un puntero (row locator) a la fila real en la tabla.

- **Puede haber múltiples** por tabla (SQL Server soporta hasta 999).
- No cambia el orden físico de las filas.
- El motor primero busca en el índice el puntero, y luego hace un **key lookup** a la fila completa (excepto si es un covering index).

```
Non-clustered index en HolderId:

Árbol B del índice:
[HolderId=A → puntero a fila con PolicyId=1]
[HolderId=A → puntero a fila con PolicyId=3]
[HolderId=B → puntero a fila con PolicyId=2]

SELECT * FROM Policies WHERE HolderId = 'A'
→ El índice encuentra los punteros → salta a las filas reales en la tabla.
```

```sql
CREATE NONCLUSTERED INDEX IX_Policies_HolderId ON Policies (HolderId);

-- Con columnas adicionales incluidas (covering index):
CREATE NONCLUSTERED INDEX IX_Policies_HolderId_Status
ON Policies (HolderId)
INCLUDE (Status, ExpiresAt);
-- Evita el key lookup para queries que solo piden HolderId, Status y ExpiresAt.
```

---

## Clustered vs Non-Clustered — Tabla comparativa

| Aspecto | Clustered | Non-Clustered |
|---|---|---|
| Cantidad por tabla | **1** | Hasta 999 (SQL Server) |
| Almacenamiento | Las filas **son** el índice | Estructura **separada** con punteros |
| Orden físico de filas | Sí, las reordena | No afecta |
| Velocidad de lectura por PK | Muy rápida (dato directo) | Requiere key lookup adicional |
| Lecturas por rango | Muy eficientes | Menos eficientes |
| Costo de escritura (INSERT/UPDATE) | Alto (reordena páginas) | Moderado |
| Caso de uso típico | PK, columnas de búsqueda principal | FKs, filtros frecuentes, columnas secundarias |

---

## Covering Index

Un covering index es un non-clustered index que **incluye todas las columnas** que necesita la query, eliminando el key lookup:

```sql
-- Query frecuente:
SELECT Status, ExpiresAt FROM Policies WHERE HolderId = 42;

-- Covering index: el índice ya tiene todo, no necesita ir a la tabla
CREATE NONCLUSTERED INDEX IX_Policies_Covering
ON Policies (HolderId)
INCLUDE (Status, ExpiresAt);
```

---

## Cuándo crear un índice

**Sí:**
- Columnas usadas frecuentemente en `WHERE`, `JOIN ON`, `ORDER BY`
- Foreign keys (SQL Server no las indexa automáticamente, PostgreSQL sí)
- Columnas con alta cardinalidad (muchos valores distintos)

**Con cuidado:**
- Tablas con muchas escrituras (INSERT/UPDATE/DELETE): cada índice adicional tiene costo de mantenimiento
- Columnas con baja cardinalidad (`bool`, `status` con 3 valores): el motor puede preferir full scan de todas formas
- No indexar columnas que raramente aparecen en filtros

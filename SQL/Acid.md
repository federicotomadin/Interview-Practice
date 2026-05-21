# ACID

**ACID** es el conjunto de cuatro propiedades que garantizan que las transacciones de base de datos se procesen de forma confiable. Una *transacción* es una unidad lógica de trabajo (una o más operaciones SQL) que debe tratarse como una operación única e indivisible.

---

## A — Atomicity

**Todo o nada.** La transacción se ejecuta completa, o no se ejecuta en absoluto. Si algo falla a mitad de camino, todos los cambios parciales se revierten (rollback).

```sql
BEGIN TRANSACTION;
    UPDATE Cuentas SET Saldo = Saldo - 1000 WHERE Id = 'A';
    UPDATE Cuentas SET Saldo = Saldo + 1000 WHERE Id = 'B';
COMMIT;
-- Si el segundo UPDATE falla, el primero también se revierte.
```

```csharp
using var transaction = await _db.Database.BeginTransactionAsync();
try
{
    cuentaA.Saldo -= 1000;
    cuentaB.Saldo += 1000;
    await _db.SaveChangesAsync();
    await transaction.CommitAsync();
}
catch
{
    await transaction.RollbackAsync();
    throw;
}
```

---

## C — Consistency

La transacción lleva la DB de un estado válido a otro estado válido. Las reglas de integridad (constraints, FKs, checks, triggers) se respetan al final de cada transacción.

```sql
ALTER TABLE Cuentas ADD CONSTRAINT CK_SaldoPositivo CHECK (Saldo >= 0);
-- Una transferencia que deje saldo negativo falla automáticamente.
```

> **Nota:** ACID consistency ≠ consistency del CAP theorem. ACID consistency = respetar reglas de integridad. CAP consistency = todos los nodos ven los mismos datos al mismo tiempo.

---

## I — Isolation

Las transacciones concurrentes no se "ven" entre sí mientras están en progreso. El resultado de ejecutar N transacciones en paralelo debe ser equivalente a ejecutarlas en algún orden secuencial.

**Niveles de aislamiento SQL:**

| Nivel | Permite |
|---|---|
| Read Uncommitted | Dirty reads (leer cambios no commiteados) |
| Read Committed | Solo lee datos commiteados (default en SQL Server) |
| Repeatable Read | Mismos valores en relecturas, pero permite phantom reads |
| Serializable | Aislamiento total, como si fueran secuenciales |

Cuanto más alto el aislamiento, más seguro pero más lento (más locks, menos concurrencia).

```csharp
using var tx = await _db.Database.BeginTransactionAsync(IsolationLevel.Serializable);
```

---

## D — Durability

Una vez commiteada, la transacción es permanente. Aunque se caiga el servidor, se corte la luz, o explote el datacenter, los datos siguen ahí.

Las DBs escriben primero a un **write-ahead log (WAL)** en disco antes de aplicar los cambios a las páginas de datos. Si el sistema se cae, al reiniciar reproduce el log y reconstruye el estado.

---

## ACID vs BASE

Las DBs NoSQL distribuidas muchas veces sacrifican parte de ACID a cambio de escalabilidad. Adoptan el modelo **BASE**:

| | ACID | BASE |
|---|---|---|
| Filosofía | Corrección > escalabilidad | Escalabilidad > corrección estricta |
| Propiedades | Atomicity, Consistency, Isolation, Durability | Basically Available, Soft state, Eventual consistency |
| Típico de | SQL Server, PostgreSQL, Oracle | Cassandra, DynamoDB, Cosmos (configurable), Mongo |

> **Cosmos DB** garantiza ACID dentro de una sola partición lógica. Cross-partition no hay transacciones ACID, por eso elegir bien la partition key es tan importante.

> Relational databases (SQL Server, PostgreSQL, Oracle, MySQL/InnoDB) are designed to be ACID-compliant. Many NoSQL databases relax some of these properties (often favoring availability and partition tolerance — see CAP theorem) although several modern NoSQL engines now support ACID transactions as well (e.g. MongoDB multi-document transactions).

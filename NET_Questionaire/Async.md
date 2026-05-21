# Async / Await y CancellationToken

## async / await

Es el modelo para escribir código asincrónico sin enredarse en callbacks. La idea fundamental: **liberar el hilo mientras esperás algo (I/O)**, para que pueda hacer otra cosa.

### Cómo funciona conceptualmente

```csharp
public async Task<Policy> GetPolicyAsync(int id)
{
    var policy = await _db.Policies.FindAsync(id);  // ① marca de pausa
    policy.LastAccessed = DateTime.UtcNow;          // ② se ejecuta DESPUÉS de que vuelva la DB
    await _db.SaveChangesAsync();
    return policy;
}
```

Cuando el código llega a `await`:

1. Si la operación ya terminó (raro), sigue sincrónicamente.
2. Si no, el método retorna un `Task` incompleto al caller, y el **hilo queda libre**.
3. Cuando la operación de I/O termina (la DB respondió), el runtime programa la continuación (la línea ②) en un hilo del pool.

> **Punto clave:** `async/await` no crea hilos. Sirve para no bloquear el hilo actual mientras se espera I/O. Es escalabilidad, no paralelismo.

### Reglas y gotchas

- **No uses `async void`** excepto en handlers de eventos. No se pueden esperar ni capturar sus excepciones.
- **No mezcles `.Result` o `.Wait()` con `await`**: deadlocks clásicos (sobre todo en contextos con `SynchronizationContext`).
- **Async all the way**: si una función de abajo es `async`, todas las de arriba también.

```csharp
// ❌ Bloquea el hilo, anula los beneficios
var policy = GetPolicyAsync(id).Result;

// ✅
var policy = await GetPolicyAsync(id);
```

### ConfigureAwait(false)

En código de librería (no en controllers de ASP.NET Core), usá `.ConfigureAwait(false)` para no forzar la continuación al contexto original:

```csharp
var data = await httpClient.GetStringAsync(url).ConfigureAwait(false);
```

En ASP.NET Core no hay `SynchronizationContext`, así que es menos crítico, pero sigue siendo buena práctica en librerías reutilizables.

---

## CancellationToken

`CancellationToken` es un mecanismo **cooperativo** de cancelación. Alguien pide cancelar, y el código en ejecución chequea periódicamente y decide cortar.

**"Cooperativo"** porque el código tiene que chequear el token; no se interrumpe mágicamente. Si tu método ignora el token, no se cancela.

### ¿Quién dispara la cancelación?

**1. Timeout explícito:**

```csharp
using var cts = new CancellationTokenSource(TimeSpan.FromSeconds(5));
await algoLargo(cts.Token);
// Si pasan 5s, el token se cancela.
```

**2. El cliente cierra la conexión HTTP** (ASP.NET Core te inyecta este token):

```csharp
[HttpGet("policies/{id}")]
public async Task<IActionResult> Get(int id, CancellationToken ct)
{
    var policy = await _db.Policies.FindAsync(new object[] { id }, ct);
    return Ok(policy);
}
```

Si el usuario navega lejos o cierra la pestaña, ASP.NET Core cancela `ct`. La query a la DB se aborta, libera la conexión, no malgasta recursos.

**3. Cancelación manual:**

```csharp
var cts = new CancellationTokenSource();
button.Click += (_, _) => cts.Cancel();
```

### Ejemplos prácticos

**Operación larga con HTTP:**

```csharp
public async Task<Report> GenerarReporteAsync(CancellationToken ct)
{
    var data = await _http.GetAsync("https://api.externa/datos", ct);
    var procesado = await _db.ProcesarAsync(ct);
    return new Report(procesado);
}
```

**Loop largo (CPU bound):**

```csharp
public void ProcesarItems(IEnumerable<Item> items, CancellationToken ct)
{
    foreach (var item in items)
    {
        ct.ThrowIfCancellationRequested();  // chequeo explícito
        Procesar(item);
    }
}
```

**Endpoint con DB + API externa:**

```csharp
[HttpGet("policies/expired")]
public async Task<IActionResult> Expired(CancellationToken ct)
{
    var policies = await _db.Policies
        .Where(p => p.ExpiresAt < DateTime.UtcNow)
        .ToListAsync(ct);

    foreach (var p in policies)
    {
        ct.ThrowIfCancellationRequested();
        await _notifier.NotifyAsync(p, ct);
    }

    return Ok();
}
```

### Patrón: "todo método async recibe un CancellationToken"

```csharp
public async Task<Policy> GetAsync(int id, CancellationToken ct = default)
public async Task SaveAsync(Policy p, CancellationToken ct = default)
public async Task<bool> ExistsAsync(int id, CancellationToken ct = default)
```

Es el equivalente de "async all the way down" pero para cancelación.

### Combinar tokens

```csharp
using var linked = CancellationTokenSource.CreateLinkedTokenSource(
    requestCt,
    new CancellationTokenSource(TimeSpan.FromSeconds(10)).Token
);

await operacion(linked.Token);
// Se cancela si el cliente se va O si pasan 10s, lo que ocurra primero.
```

### Por qué importa en producción

Sin cancellation tokens: un cliente que se va deja al servidor procesando una query de 30 segundos para nada. Bajo carga, se malgastan CPU, conexiones a DB, memoria, en respuestas que nadie va a leer.

Con cancellation tokens bien propagados, esos recursos se liberan en cuanto el cliente desaparece.

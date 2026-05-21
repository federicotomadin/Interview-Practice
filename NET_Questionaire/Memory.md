# Memory Management in .NET

## Heap y Stack

En .NET la memoria de un proceso se organiza en dos regiones principales: **stack** y **heap**.

### Stack

Estructura LIFO (Last In, First Out) que vive por hilo. Es rapidísima porque solo mueve un puntero hacia arriba/abajo. Almacena:

- Variables locales de tipos por valor (`int`, `bool`, `struct`, etc.)
- Parámetros de métodos
- Direcciones de retorno
- Referencias (los punteros) a objetos del heap

Cuando un método termina, su "stack frame" se libera automáticamente. **No interviene el GC.**

### Heap

Región grande y dinámica donde viven los objetos por referencia (`class`, `string`, arrays, etc.). Es más lento de asignar y requiere gestión.

```csharp
public void Ejemplo()
{
    int edad = 30;                    // stack
    Persona p = new Persona("Fede");  // 'p' (referencia) en stack, objeto Persona en heap
}
// Al salir del método: 'edad' y 'p' se borran del stack automáticamente.
// El objeto Persona queda en el heap hasta que el GC lo libere.
```

---

## Garbage Collector (GC)

El GC libera memoria del heap cuando los objetos ya no son alcanzables (no hay referencias activas). Funciona por **generaciones**:

| Generación | Descripción | Frecuencia |
|---|---|---|
| **Gen 0** | Objetos recién creados | Muy seguido, barato |
| **Gen 1** | Sobrevivieron una colección de Gen 0 | Buffer entre 0 y 2 |
| **Gen 2** | Objetos de larga vida (caches, singletons) | Poco, caro |
| **LOH** | Objetos > 85 KB. Se trata aparte porque comprimirlos es caro | Junto con Gen 2 |

**La idea:** la mayoría de los objetos mueren jóvenes, así que limpiar solo Gen 0 ya recupera mucha memoria.

### Implicancias prácticas

En código de alto rendimiento (Azure Functions, APIs con mucho throughput) conviene evitar asignaciones innecesarias en el heap. Por eso existen `Span<T>`, `stackalloc`, `ArrayPool<T>`, y por eso `struct` puede ser preferible a `class` para tipos chicos e inmutables.

---

## Value Type vs Reference Type

| Aspecto | Value Type | Reference Type |
|---|---|---|
| Ejemplos | `int`, `bool`, `decimal`, `DateTime`, `struct`, `enum` | `class`, `string`, array, delegate, interface |
| Dónde vive | Stack (o inline dentro de un objeto del heap) | Heap (la referencia en stack) |
| Asignación (`=`) | Copia el valor completo | Copia la referencia (apuntan al mismo objeto) |
| Default | Valor cero (`0`, `false`, struct con campos default) | `null` |
| Comparación `==` | Compara valor | Compara referencia (salvo overload, como en `string`) |

### Ejemplo clave

```csharp
// VALUE TYPE
int a = 5;
int b = a;   // copia el valor
b = 10;
Console.WriteLine(a); // 5  ← 'a' no cambió

// REFERENCE TYPE
var lista1 = new List<int> { 1, 2, 3 };
var lista2 = lista1;   // copia la referencia, no la lista
lista2.Add(4);
Console.WriteLine(lista1.Count); // 4  ← lista1 también "cambió"
```

### Paso a métodos

Por default todo se pasa **por valor** (lo que se copia es el valor o la referencia). Para pasar la variable en sí se usa `ref`, `in`, `out`:

```csharp
void NoMutar(Persona p) { p = new Persona("Otro"); }     // no afecta al caller
void SiMutar(ref Persona p) { p = new Persona("Otro"); } // afecta al caller
void Leer(in Persona p) { /* readonly, evita copia en structs grandes */ }
```

### Cuidado con structs

Si hacés un `struct` grande y lo pasás por valor en loops, hay copias costosas. Regla de oro: **structs chicos (≤16 bytes), inmutables, sin lógica compleja**. Para todo lo demás, `class`.

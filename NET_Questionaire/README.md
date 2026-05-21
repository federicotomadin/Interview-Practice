## Basic .NET Knowledge

* [Basic .NET Knowledge](#basic-net-knowledge)
* [C# Language Proficiency](#c-language-proficiency)
* [ASP.NET and Web Development](#aspnet-and-web-development)
* [Database and Entity Framework](#database-and-entity-framework)
* [Dependency Injection and IoC Containers](#dependency-injection-and-ioc-containers)
* [Testing and Debugging](#testing-and-debugging)
* [RESTful API Development](#restful-api-development)
* [Performance Optimization](#performance-optimization)
* [Security](#security)
* [Version Control and DevOps](#version-control-and-devops)
* [Design Patterns and Best Practices](#design-patterns-and-best-practices)
* [Dependency Injection and IoC Containers](#dependency-injection-and-ioc-containers)
* [Inversion of Control IoC](#what-is-inversion-of-control-ioc)
* [Deletegates](#delegates)
* [FuncAndAction](#func-and-action)
* [Lazy loading in Entity Framework](#lazy-loading-in-entity-framework)
* [IQueryable vs IEnumerable](#iqueryable-vs-ienumerable)


## Basic .NET Knowledge
- **Q: What is .NET and what is its role in software development?**
    - .NET is a software framework developed by Microsoft that provides a platform for building and running applications across different operating systems. Its role in software development is to simplify the process of creating robust, secure, and scalable applications.

- **Q: Explain the Common Language Runtime (CLR) and its significance in .NET.**
    - The CLR is a key component of the .NET Framework responsible for executing and managing applications written in different programming languages. It provides services like memory management, exception handling, security, and type safety, ensuring efficient and secure execution of applications.

- **Q: What is the CLR in more detail and what services does it provide?**
    - The **Common Language Runtime (CLR)** is the virtual machine component of .NET that runs managed code. When you compile C# (or VB.NET, F#, etc.), the compiler produces **Intermediate Language (IL)** code plus metadata, packaged into an assembly (`.dll` / `.exe`). At runtime the CLR loads that assembly and uses a **JIT (Just-In-Time) compiler** to translate IL into native machine code for the CPU it is running on.
    - Key services provided by the CLR:
        - **Memory management** via the **Garbage Collector (GC)**, which automatically allocates and reclaims memory.
        - **Type safety** — the CLR verifies that code does not access memory it shouldn't.
        - **Exception handling** in a unified way across languages.
        - **Security** through Code Access Security and verification of IL.
        - **Thread management** and synchronization primitives.
        - **Interoperability** with native (unmanaged) code via P/Invoke and COM interop.
        - **Cross-language integration** — any .NET language compiles to IL, so they can call each other seamlessly.

- **Q: What is the difference between managed and unmanaged code in .NET?**
    - **Managed code** is code that runs **under the control of the CLR**. Languages like C#, VB.NET and F# compile to **Intermediate Language (IL)**, and the CLR provides services on top of it: memory allocation, **garbage collection**, type safety, exception handling, security checks, and JIT compilation to native code. The developer does not manage memory manually — the GC reclaims unused objects automatically.
    - **Unmanaged code** is code that runs **directly against the operating system**, outside the CLR. Examples include code written in C/C++, Win32 API calls, COM components, and most native libraries. Unmanaged code is compiled directly to machine code and is responsible for its **own memory management** (`malloc`/`free`, `new`/`delete`), error handling, and security. There is no GC and no IL verification.
    - Key differences:

| Aspect | Managed Code | Unmanaged Code |
| --- | --- | --- |
| Execution environment | Runs inside the CLR | Runs directly on the OS |
| Compilation | Source → IL → JIT → native | Source → native machine code |
| Memory management | Automatic (Garbage Collector) | Manual (developer's responsibility) |
| Type safety | Enforced by the CLR | Not enforced |
| Exception handling | Unified .NET exceptions | OS / language specific (e.g. SEH, errno) |
| Performance | Slightly slower due to runtime services, but predictable | Potentially faster, less overhead |
| Portability | Portable across platforms supported by .NET | Tied to a specific OS / architecture |
| Examples | C#, VB.NET, F# | C, C++, Win32 API, COM |

- **Interop**: .NET can call into unmanaged code using **P/Invoke** (`[DllImport]`) or **COM interop**. When this happens, the runtime *marshals* data between managed and unmanaged memory. Types like `IntPtr`, `SafeHandle` and the `IDisposable` pattern (with finalizers) exist precisely to bridge the two worlds and make sure unmanaged resources (file handles, sockets, native memory) are released deterministically, since the GC does not track them.

```csharp
using System.Runtime.InteropServices;

class NativeBridge
{
    // Calling unmanaged code (Win32 API) from managed C#
    [DllImport("user32.dll", CharSet = CharSet.Unicode)]
    private static extern int MessageBox(IntPtr hWnd, string text, string caption, uint type);

    public static void Show() => MessageBox(IntPtr.Zero, "Hello", "Demo", 0);
}
```

## C# Language Proficiency
- **Q: Describe the differences between value types and reference types in C#.**
    - Value types hold the actual data and are stored on the stack, while reference types hold a reference to the data and are stored on the heap. Value types include primitive types and structures, while reference types include classes, interfaces, and delegates.

- **Q: What are nullable types, and when would you use them?**
    - Nullable types allow variables to have a value or be null. They are useful when dealing with scenarios where a value may or may not be present, such as database fields or optional parameters.

```csharp
int? nullableInt = null;
int nonNullableInt = 10;

Console.WriteLine(nullableInt); // Output: (null)
Console.WriteLine(nullableInt.GetValueOrDefault()); // Output: 0

Console.WriteLine(nonNullableInt); // Output: 10
```

- **Q: Explain the concept of delegates and events in C#.**
    - Delegates are type-safe function pointers that allow methods to be passed as parameters or stored as variables. Events are a special type of delegates used for implementing the publisher-subscriber pattern, allowing objects to communicate and notify each other of specific actions or state changes.

```csharp
public class EventPublisher
{
    public delegate void EventHandler(string message);

    public event EventHandler MessageSent;

    public void SendMessage(string message)
    {
        MessageSent?.Invoke(message);
    }
}

public class EventSubscriber
{
    public EventSubscriber(EventPublisher publisher)
    {
        publisher.MessageSent += HandleMessage;
    }

    public void HandleMessage(string message)
    {
        Console.WriteLine("Received message: " + message);
    }
}

// Usage
EventPublisher publisher = new EventPublisher();
EventSubscriber subscriber = new EventSubscriber(publisher);

publisher.SendMessage("Hello World!"); // Output: Received message: Hello World!
```

- **Q: What is the difference between `const` and `readonly` in C#?**
    - Both are used to declare values that should not change, but they behave very differently:
        - **`const`** — *compile-time constant*. The value must be assigned at declaration and must be a literal known at compile time. It is **implicitly `static`** (belongs to the type, not an instance) and its value is **baked into the calling assembly** at compile time. If you change a `const` in a referenced library, every consumer must be **recompiled** to see the new value. Only primitive types, `string`, and `null` references can be `const`.
        - **`readonly`** — *runtime constant*. The field can be assigned **only** at declaration or inside a constructor (instance constructor for instance fields, static constructor for `static readonly`). Its value is resolved at runtime, so changes in a referenced library are picked up without recompiling consumers. Works with **any type**, including reference types and structs.
    - Rule of thumb: use `const` for truly fixed values (e.g. `const double Pi = 3.14159;`), and `readonly` for values that depend on runtime context (e.g. configuration, dependencies, computed values).

```csharp
public class Circle
{
    public const double Pi = 3.14159;          // compile-time constant
    public readonly double Radius;             // assigned in constructor
    public static readonly DateTime CreatedAt; // assigned in static constructor

    static Circle()
    {
        CreatedAt = DateTime.UtcNow;
    }

    public Circle(double radius)
    {
        Radius = radius; // allowed: inside constructor
    }
}
```

- **Q: Explain the `ref` and `out` keywords in C#.**
    - Both keywords cause arguments to be passed **by reference** instead of by value, meaning the called method can modify the caller's variable. The difference is in the contract about initialization:
        - **`ref`** — the variable **must be initialized before** being passed to the method. The method may read the existing value and may modify it. Used when the method needs both an input and an output through the same parameter.
        - **`out`** — the variable does **not** need to be initialized before the call, but the method **must assign** a value to it before returning. Used when the method's purpose is to produce one or more outputs through parameters (e.g. `int.TryParse`).
    - Both must be specified at the call site as well as in the method signature, and overload resolution treats `ref` and `out` as different signatures.

```csharp
// ref example: caller initializes, method modifies in place
public static void Increment(ref int value)
{
    value++;
}

int x = 5;
Increment(ref x);
Console.WriteLine(x); // 6

// out example: method must assign before returning
public static bool TryDivide(int a, int b, out int result)
{
    if (b == 0) { result = 0; return false; }
    result = a / b;
    return true;
}

if (TryDivide(10, 2, out int q))
    Console.WriteLine(q); // 5
```

- **Q: What is the difference between `==` and `Equals()` in C#?**
    - `==` is an **operator** resolved at **compile time** based on the static (declared) type of its operands. `Equals()` is a **virtual method** on `object` resolved at **runtime** based on the actual type of the instance.
    - For **value types**, `==` (when defined) and `Equals()` typically compare values, and both behave the same.
    - For **reference types**, by default `==` does a **reference comparison** (are they the same object on the heap?) and `Equals()` also does a reference comparison **unless** the type overrides it (e.g. `string` overrides `Equals` *and* overloads `==` to compare characters).
    - Because `==` is statically dispatched, it can give "surprising" results when called through `object` references — the operator chosen is the one for `object`, not for the runtime type.
    - For safe, **null-tolerant**, value-based equality regardless of static type, prefer `object.Equals(a, b)` or `EqualityComparer<T>.Default.Equals(a, b)`.

```csharp
string a = "hello";
string b = string.Concat("hel", "lo");

Console.WriteLine(a == b);        // True  -> string overloads ==
Console.WriteLine(a.Equals(b));   // True  -> string overrides Equals

object oa = a;
object ob = b;
Console.WriteLine(oa == ob);      // May be False -> uses object's reference ==
Console.WriteLine(oa.Equals(ob)); // True  -> virtual call dispatches to string.Equals
```

## ASP.NET and Web Development
- **Q: What is ASP.NET, and how does it differ from ASP.NET Core?**
    - ASP.NET is a web development framework that allows building web applications using .NET. ASP.NET Core is the next iteration of ASP.NET, a cross-platform framework that is lightweight, modular, and optimized for modern web development.

- **Q: Explain the MVC (Model-View-Controller) architecture in ASP.NET.**
    - MVC is an architectural pattern for designing web applications. In ASP.NET MVC, the model represents the data, the view handles the presentation, and the controller manages the flow and interacts with both the model and view.

- **Q: Discuss the differences between ASP.NET Web Forms and ASP.NET MVC.**
    - ASP.NET Web Forms is an older framework that provides a more event-driven and stateful approach to web development. ASP.NET MVC, on the other hand, follows the MVC pattern, promotes separation of concerns, and provides more control over the HTML and client-side behavior.

### Filters vs Middleware

Son dos mecanismos distintos para interceptar el request. Es común confundirlos.

**Middleware** es el pipeline general del request. Cada middleware decide si ejecutar el siguiente o cortar. Se ejecuta para **todos los requests** (HTTP, no solo MVC).

```csharp
// Program.cs
app.UseHttpsRedirection();
app.UseAuthentication();
app.UseAuthorization();
app.UseMiddleware<LoggingMiddleware>();
app.MapControllers();
```

Un middleware custom:

```csharp
public class LoggingMiddleware
{
    private readonly RequestDelegate _next;
    public LoggingMiddleware(RequestDelegate next) => _next = next;

    public async Task InvokeAsync(HttpContext ctx, ILogger<LoggingMiddleware> log)
    {
        var sw = Stopwatch.StartNew();
        await _next(ctx);
        log.LogInformation("{Path} took {Ms}ms", ctx.Request.Path, sw.ElapsedMilliseconds);
    }
}
```

**Filters** son específicos del pipeline de MVC/API controllers. Se ejecutan después del routing y conocen el contexto MVC (action, model state, result).

| Filtro | Cuándo corre | Uso típico |
|---|---|---|
| `IAuthorizationFilter` | Antes de todo | Autorización custom |
| `IResourceFilter` | Antes/después del model binding | Caching |
| `IActionFilter` | Antes/después del action | Logging de acción, validación |
| `IExceptionFilter` | Si el action lanza | Manejo global de excepciones |
| `IResultFilter` | Antes/después de ejecutar el result | Transformar respuesta |

```csharp
public class ValidateModelAttribute : ActionFilterAttribute
{
    public override void OnActionExecuting(ActionExecutingContext context)
    {
        if (!context.ModelState.IsValid)
            context.Result = new BadRequestObjectResult(context.ModelState);
    }
}

[ValidateModel]
public async Task<IActionResult> Create([FromBody] CreatePolicyDto dto) { ... }
```

**Cuándo usar cuál:**
- **Middleware:** lógica transversal a toda la app (logging, CORS, compresión, auth, rate limiting).
- **Filter:** lógica que depende del contexto MVC (validar ModelState, transformar resultado, manejar excepciones de la API).

**Orden de ejecución:**
```
Request → Middleware (entrada) → Routing → Filters (entrada)
       → Action → Filters (salida) → Middleware (salida) → Response
```

## Database and Entity Framework
- **Q: How do you connect a .NET application to a database?**
    - ADO.NET provides various classes and libraries to connect to databases. Developers can use classes like SqlConnection and SqlCommand to establish connections and execute queries against the database.

- **Q: Explain the Entity Framework and its role in data access in .NET.**
    - Entity Framework is an Object-Relational Mapping (ORM) framework that simplifies data access in .NET applications. It allows developers to work with databases using object-oriented techniques, abstracting away the underlying database details.

- **Q: What is Code-First vs. Database-First development in Entity Framework?**
    - Code-First development involves creating the object model first and then generating the database schema based on the model. Database-First development, on the other hand, involves creating the database schema first and then generating the object model based on the schema.

## Dependency Injection and IoC Containers
- **Q: What is dependency injection, and why is it important in .NET development?**
    - Dependency injection is a design pattern that promotes loose coupling and modular design by allowing dependencies to be injected into an object from external sources. It improves testability, maintainability, and flexibility of the code.

- **Q: Can you name some popular IoC (Inversion of Control) containers used in .NET?**
    - Some popular IoC containers in .NET include Autofac, Ninject, Unity, and Simple Injector. These frameworks help manage object creation, lifetime, and injection of dependencies in a .NET application.

# .NET Interview Questions and Answers

This readme.md file provides a list of .NET interview questions along with their corresponding answers. Each section focuses on a specific topic in .NET development.


## Testing and Debugging

**Q: How do you perform unit testing in .NET applications?**

A: Unit testing in .NET applications involves writing test methods to verify the behavior of individual units of code (such as methods or classes). Popular unit testing frameworks in .NET include NUnit, MSTest, and xUnit. These frameworks provide assertions, test runners, and other utilities to help write and execute unit tests. Unit tests should be independent, isolated, and cover different scenarios to ensure the correctness of the code.

### Principio FIRST (buenos tests unitarios)

Acrónimo popularizado por Robert C. Martin (Clean Code) para tests unitarios de calidad:

| Letra | Principio | Descripción |
|---|---|---|
| **F** | **Fast** | Deben correr en milisegundos. Si tardan, no los corrés seguido. |
| **I** | **Independent / Isolated** | Ningún test depende del orden ni del resultado de otro. |
| **R** | **Repeatable** | Corren igual en cualquier entorno. Sin depender de fecha, API externa o red. |
| **S** | **Self-validating** | El test mismo dice "pasó" o "falló". Sin revisar logs manualmente. |
| **T** | **Timely** | Se escriben junto al código de producción (TDD), no semanas después. |

Si un test falla alguno de estos, suele convertirse en un test que se ignora o se borra.

### Pirámide de Testing

Guía de cuántos tests de cada tipo tener, propuesta por Mike Cohn:

```
        /\
       /  \      ← E2E / UI (pocos, lentos, frágiles)
      /----\
     /      \    ← Integración (moderados)
    /--------\
   /          \  ← Unitarios (muchísimos, rápidos)
  /____________\
```

- **Base (unit tests):** muchísimos. Validan una unidad aislada. Rápidos, baratos, deterministas. Usan mocks para dependencias.
- **Medio (integration tests):** menos. Validan que varios componentes funcionen juntos (endpoint + DB en memoria, repository + SQL).
- **Cima (E2E / UI):** pocos. Validan flujos completos. Lentos, frágiles, caros de mantener. Solo para flujos críticos del negocio.

**La intuición:** si un test E2E falla, no sabés qué se rompió (¿UI? ¿API? ¿DB?). Si un unit test falla, sabés exactamente qué clase y método.

**Antipatrones comunes:**
- **Ice cream cone** (pirámide invertida): muchos E2E, pocos unit. Suite lenta, builds rojos seguido.
- **Hourglass:** muchos unit y muchos E2E, sin integración. Pierde la capa que valida contratos entre componentes.

**Q: What is the purpose of the Debugger class in C#?**

A: The Debugger class in C# provides a set of static methods and properties that assist in debugging applications. It allows developers to control program execution, set breakpoints, examine variables, and step through code during runtime. The Debugger class is primarily used for debugging purposes and is often utilized in conjunction with integrated development environments (IDEs) like Visual Studio.

**Q: Explain the differences between debugging and tracing.**

A: Debugging and tracing are techniques used for diagnosing issues and understanding the behavior of an application, but they serve different purposes.

Debugging:
- Debugging is the process of identifying and fixing errors or defects in code.
- It involves stepping through the code, setting breakpoints, examining variables, and analyzing the program's state during runtime.
- Debugging is an interactive process aimed at finding and resolving issues that cause the application to behave incorrectly or crash.
- It is primarily used during development or testing phases.

Tracing:
- Tracing is the process of capturing information about the execution of an application for analysis or monitoring purposes.
- It involves logging messages or events at various points in the code to track the flow and behavior of the application.
- Tracing provides insights into the sequence of operations, performance bottlenecks, and potential issues in production environments.
- It is useful for troubleshooting, performance optimization, and auditing purposes.

While debugging is focused on identifying and fixing issues during development, tracing provides a broader picture of an application's execution for analysis and monitoring purposes in production environments.

## RESTful API Development

**Q: How do you create a RESTful API in .NET using ASP.NET Web API or ASP.NET Core?**

A: To create a RESTful API in .NET using ASP.NET Web API or ASP.NET Core, you need to define endpoints, create controllers, implement actions for HTTP verbs, model data, configure routing, and apply authentication/authorization.

**Q: What is Swagger, and how can it be useful when building APIs?**

A: Swagger is a framework for designing, building, documenting, and consuming RESTful APIs. It generates interactive documentation, enables code generation, supports testing and mocking, and facilitates API exploration. It helps developers and consumers understand, test, and interact with APIs effectively.

## Performance Optimization

**Q: What strategies would you use to optimize the performance of a .NET application?**

A: Strategies for optimizing the performance of a .NET application include using efficient algorithms and data structures, implementing caching mechanisms, utilizing asynchronous programming, profiling the code, optimizing database queries, efficient resource utilization, load balancing and scaling, and performing performance testing.

**Q: Can you explain the concept of caching in .NET?**

A: Caching in .NET involves storing frequently accessed data in memory to improve performance. It can be implemented using in-memory cache providers like `MemoryCache` or distributed cache providers like `RedisCache`. Caching reduces the need for repeated expensive computations or database queries by retrieving data from the cache instead.

## Security

**Q: How do you handle authentication and authorization in a .NET application?**

A: Authentication verifies the identity of users, while authorization determines what actions they are allowed to perform. In a .NET application, you can handle authentication and authorization using various methods, such as ASP.NET Identity, JWT (JSON Web Tokens), OAuth, or custom authentication/authorization mechanisms. These methods involve validating user credentials, generating and validating tokens, managing roles and permissions, and securing sensitive data.

**Q: What are some common security vulnerabilities in web applications, and how can they be mitigated in a .NET context?**

A: Some common security vulnerabilities in web applications include cross-site scripting (XSS), cross-site request forgery (CSRF), SQL injection, and insecure direct object references. To mitigate these vulnerabilities in a .NET context, you can use measures like input validation and sanitization, parameterized queries or ORM tools to prevent SQL injection, enforcing secure coding practices, implementing output encoding, using anti-forgery tokens to prevent CSRF attacks, implementing secure authentication and authorization mechanisms, and keeping the application and its dependencies up to date with security patches.

## Version Control and DevOps

**Q: Have you worked with version control systems like Git in a team setting?**

A: Yes, I have worked with version control systems like Git in a team setting. I am familiar with Git workflows, branching strategies, merging, resolving conflicts, and collaborating with other team members using Git-based tools like GitHub, GitLab, or Bitbucket.

**Q: What is Continuous Integration (CI) and Continuous Deployment (CD), and how do they apply to .NET development?**

A: Continuous Integration (CI) is a software development practice where developers regularly merge their code changes to a shared repository. This process involves automatically building and testing the code to identify integration issues early. Continuous Deployment (CD) is the automated release of software to production or other environments after successful CI. In .NET development, tools like Azure DevOps, Jenkins, or TeamCity can be used to set up CI/CD pipelines. These pipelines automatically build, test, and deploy .NET applications, ensuring faster and more reliable software delivery.

## Design Patterns and Best Practices

**Q: Can you name and describe some design patterns commonly used in .NET development?**

A: Some commonly used design patterns in .NET development include:

1. **Singleton**: Ensures a class has only one instance and provides a global point of access to it.
2. **Factory**: Provides an interface for creating objects without specifying their concrete classes.
3. **Repository**: Encapsulates the logic for retrieving, storing, and querying data from a data source.
4. **Observer**: Defines a one-to-many dependency between objects, where a change in one object triggers updates in dependent objects.
5. **Adapter**: Converts the interface of a class into another interface that clients expect.
6. **Strategy**: Defines a family of algorithms, encapsulates each one, and makes them interchangeable at runtime.
7. **Decorator**: Allows behavior to be added to an object dynamically, without affecting the behavior of other objects.

These patterns, along with others like MVC, MVVM, and Dependency Injection, help improve code modularity, extensibility, and maintainability in .NET applications.

**Q: What are some coding best practices you follow when writing .NET code?**

A: Some coding best practices in .NET development include:

1. Following naming conventions for classes, methods, and variables to improve code readability.
2. Writing self-explanatory code with proper comments and documentation.
3. Using meaningful and intention-revealing names for variables and methods.
4. Applying SOLID principles to design classes and components.
5. Writing unit tests to ensure code correctness and maintainability.
6. Implementing exception handling to handle errors gracefully.
7. Avoiding code duplication by using DRY (Don't Repeat Yourself) principle.
8. Using object-oriented principles like encapsulation, inheritance, and polymorphism effectively.
9. Following coding standards and guidelines set by the .NET community, like Microsoft's C# Coding Conventions.

These practices help in producing clean, maintainable, and efficient .NET code.

# Object Lifetimes in Dependency Injection

When you register a service with an IoC container, you also specify its lifetime. This controls how long an instance of the service will live. There are three main lifetimes:

1. **Transient**:
  - A new instance of the service is created each time it is requested.
  - Suitable for lightweight, stateless services.
  - Example: If a service is registered as transient, every time a request is made for this service, a new instance is created.

2. **Scoped**:
  - A new instance of the service is created once per request (or scope).
  - Ideal for services that should be shared within a request but not beyond (e.g., database contexts in a web application).
  - In ASP.NET Core, a scope is typically tied to a single HTTP request.

3. **Singleton**:
  - A single instance of the service is created and shared throughout the application’s lifetime.
  - Suitable for services that need to maintain state or are expensive to create.
  - Be cautious with singleton services in multi-threaded environments, as they need to be thread-safe.


# What is Inversion of Control (IoC)?

Inversion of Control (IoC)
refers to the process where the control of object creation and the management of dependencies is transferred from the class itself to an external component or framework.
In simpler terms, instead of a class creating its own dependencies (e.g., via `new` keyword),
an external container or framework is responsible for providing those dependencies.

## Without IoC

A class might create its own dependencies like this:

```csharp
public class Car
{
    private readonly Engine _engine;

    public Car()
    {
        _engine = new Engine(); // The Car class is responsible for creating an Engine instance
    }
}
```

With IoC
The class receives its dependencies from an external source:

```csharp
public class Car
{
private readonly Engine _engine;

    // The dependency is injected into the class through the constructor
    public Car(Engine engine)
    {
        _engine = engine;
    }
}
```


## Delegates

A delegate in C# is a type that represents references to methods with a specific parameter list and return type. Delegates are similar to function pointers in C++ but are type-safe and object-oriented. They allow methods to be passed as parameters, making them a fundamental part of event handling and callback mechanisms in C#.


- **Type Safety**: Delegates ensure that the method signature matches the delegate’s signature, preventing runtime errors.
- **Multicast**: Delegate
- 
```csharp
// Define a delegate that takes an integer and returns void
public delegate void PrintDelegate(int number);

class Program
{
    // A method that matches the signature of the delegate
    public static void PrintNumber(int num)
    {
        Console.WriteLine("Number: " + num);
    }

    public static void Main()
    {
        // Instantiate the delegate and assign a method
        PrintDelegate print = new PrintDelegate(PrintNumber);

        // Invoke the delegate
        print(100);
    }
}
```
## Usage of Delegates:

- **Event Handling**: Delegates are the backbone of events in C#. When an event is triggered, delegates are used to call the event handler methods.
- **Callbacks**: Delegates allow asynchronous calls and callbacks to be set up, where the delegate can be invoked when a long-running task completes.
- **LINQ and Functional Programming**: Delegates are used extensively in LINQ (Language-Integrated Query) and for functional programming patterns using lambda expressions.

## Types of Delegates:

- **Single-Cast Delegate**: Represents a single method.
- **Multi-Cast Delegate**: Can hold references to multiple methods and invokes them in order when called.
- **Generic Delegates**: `Action`, `Func`, and `Predicate` are predefined delegates in .NET that are used for common scenarios:
  - **Action**: Represents a method that returns void.
  - **Func**: Represents a method that returns a value.
  - **Predicate**: Represents a method that returns a boolean.

## Func and Action

- **Func**: Represents a method that returns a value. It can take up to 16 input parameters and returns the result.
- **Action**: Represents a method that returns void. It can take up to 16 input parameters but does not return a value.

```csharp
// Func example: Takes two integers and returns their sum
Func<int, int, int> add = (a, b) => a + b;
int result = add(10, 20); // result = 30

// Action example: Prints a message to the console
Action<string> printMessage = message => Console.WriteLine(message);
printMessage("Hello, World!"); // Output: Hello, World!
```

## Lazy loading in Entity Framework

Lazy loading is the process where an entity or collection of entities is automatically loaded from the database the 
first time that a property referring to the entity/entities is accessed. 
When using POCO entity types, lazy loading is achieved by creating instances of derived proxy types and then overriding virtual 
properties to add the loading hook. Lazy loading is enabled by default in Entity Framework.

```csharp
public class BloggingContext : DbContext
{
    public BloggingContext()
    {
        this.Configuration.LazyLoadingEnabled = false;
    }

    public DbSet<Blog> Blogs { get; set; }
    public DbSet<Post> Posts { get; set; }
}
```

## IEnumerable vs ICollection vs IQueryable

La jerarquía relevante:

```
IEnumerable<T>           → iteración (foreach)
   └── ICollection<T>    → + Count, Add, Remove, Contains
        └── IList<T>     → + indexador [i], Insert, RemoveAt
```

### IEnumerable\<T\>

Lo mínimo. Solo garantiza que podés iterar con `foreach`. No conoce el tamaño sin recorrerla, y puede ser **diferida (lazy)**.

```csharp
IEnumerable<int> numeros = Enumerable.Range(1, 1_000_000)
                                     .Where(n => n % 2 == 0);
// Acá no se ejecutó nada todavía. Se ejecuta cuando iterás.
```

Esto es **lazy evaluation**: en LINQ podés encadenar `.Where().Select().Take(10)` y solo procesa lo necesario.

### ICollection\<T\>

Agrega `Count`, `Add`, `Remove`, `Contains`, `Clear`, `IsReadOnly`. Implica que la colección ya está **materializada en memoria**.

### Tradeoffs prácticos

```csharp
// ✅ Acepta cualquier cosa iterable, incluso queries diferidos
public decimal SumarPrecios(IEnumerable<Producto> productos) =>
    productos.Sum(p => p.Precio);

// ⚠️ Pide más capacidades (Count, Add). Solo si las necesitás.
public void Procesar(ICollection<Producto> productos)
{
    if (productos.Count == 0) return;
    productos.Add(new Producto());
}
```

**Cuidado con devolver `IEnumerable` desde un DbContext** — el consumidor podría enumerar dos veces y disparar dos queries a la DB:

```csharp
// ⚠️ Problema clásico con EF Core
public IEnumerable<Policy> GetPolicies() =>
    _db.Policies.Where(p => p.Active);  // todavía es IQueryable

var policies = GetPolicies();
var count = policies.Count();          // query 1
foreach (var p in policies) { ... }   // query 2 ← otra ida a la DB
```

**Regla rápida:**
- ¿Solo vas a iterar una vez? → `IEnumerable<T>`
- ¿Necesitás `Count`, `Add`, modificar? → `ICollection<T>` o `IList<T>`
- ¿Acceso por índice frecuente? → `IList<T>` o `List<T>` directo
- ¿Lectura intensiva, sin mutar? → `IReadOnlyList<T>` / `IReadOnlyCollection<T>`

### Lazy evaluation: ¿cuándo se ejecuta?

**Operadores diferidos** (no ejecutan, devuelven otro `IEnumerable`):
`.Where()`, `.Select()`, `.Take()`, `.Skip()`, `.OrderBy()`, `.GroupBy()`, `.Distinct()`

**Operadores terminales** (disparan la iteración):
```csharp
foreach (var item in query) { }     // dispara
.ToList()  .ToArray()  .ToDictionary()  // materializan
.Count()   .Sum()   .First()   .Any()   // agregan/cortan
```

**El "gotcha" clásico: enumerar dos veces:**
```csharp
var query = _db.Policies.Where(p => p.Active);
var count = query.Count();        // ← Query #1 a la DB
foreach (var p in query) { ... }  // ← Query #2 a la DB

// Solución:
var list = query.ToList();        // una sola query
var count = list.Count;           // sin viajar a DB
```

### IEnumerable\<T\> vs IQueryable\<T\>

- `IEnumerable<T>.Where(Func<T, bool>)` → el delegado se ejecuta **en memoria** (LINQ to Objects).
- `IQueryable<T>.Where(Expression<Func<T, bool>>)` → el delegado se **traduce a SQL** (LINQ to Entities).

```csharp
IEnumerable<int> numbers = new List<int> { 1, 2, 3, 4, 5 };
var filteredNumbers = numbers.Where(n => n > 2); // ejecuta en memoria

IQueryable<int> queryableNumbers = dbContext.Numbers;
var filteredQueryableNumbers = queryableNumbers.Where(n => n > 2); // genera SQL
```

## Hash Table

Una hash table es una estructura clave-valor con búsqueda en **O(1) promedio**. Cuando guardás un par (clave, valor), se calcula `hashCode = clave.GetHashCode()`, y ese hash determina en qué "bucket" del array interno se guarda.

```
clave "fede" → GetHashCode() → 8472639 → 8472639 % 16 = bucket 7
```

### Colisiones

Dos claves distintas pueden caer al mismo bucket. .NET resuelve esto con **chaining**: cada bucket es una lista enlazada de entradas. Por eso también hace falta `Equals`: el hash te lleva al bucket, `Equals` confirma cuál es la entrada correcta.

> **Regla de oro:** si overrideás `Equals`, tenés que overridear `GetHashCode`. Si no, todo se rompe en diccionarios y sets.

### En .NET

- `Dictionary<TKey, TValue>` — la implementación moderna, genérica, recomendada.
- `Hashtable` — clase vieja, no genérica (boxea value types), evitala.
- `HashSet<T>` — hash table sin valores, solo claves únicas.
- `ConcurrentDictionary<TKey, TValue>` — thread-safe.

```csharp
var precios = new Dictionary<string, decimal>
{
    ["pan"] = 1500m,
    ["leche"] = 1200m
};

if (precios.TryGetValue("pan", out var precio))   // O(1)
    Console.WriteLine(precio);
```

**Cuándo usarla:** lookups frecuentes por clave. Si tenés una `List<Producto>` y hacés `.FirstOrDefault(p => p.Id == id)` muchas veces, eso es O(n) por búsqueda. Convertirlo a `Dictionary<int, Producto>` te da O(1).

---

## Func vs Expression\<Func\>

### La diferencia fundamental

```csharp
Func<Policy, bool> func = p => p.Active;
// El compilador compila esto a IL. Es código ejecutable.
// Solo podés hacer: func(policy) → bool

Expression<Func<Policy, bool>> expr = p => p.Active;
// El compilador genera un árbol de expresión inspeccionable.
// Podés inspeccionarlo, modificarlo, o traducirlo a SQL.
```

### Cómo lo usa EF Core

`DbSet<T>` implementa `IQueryable<T>`, cuya firma de `Where` es:
```csharp
IQueryable<T> Where<T>(this IQueryable<T> source, Expression<Func<T, bool>> predicate)
```

El compilador convierte la lambda en un árbol de expresión, que EF recorre para generar SQL:
```sql
SELECT * FROM Policies WHERE Active = 1 AND Holder = 'Fede'
```

Si pasás un `Func` ya compilado a `IQueryable`, EF Core 3+ **tira excepción** (en 2.x evaluaba en cliente silenciosamente, trayendo toda la tabla).

### Cuándo usar cada uno

**Usá `Func<T, bool>` cuando:**
- Trabajás con `IEnumerable<T>` en memoria (LINQ to Objects).
- No hay traducción a SQL.
- Querés pasar comportamiento a una función propia.

**Usá `Expression<Func<T, bool>>` cuando:**
- Querés que un proveedor lo traduzca (EF Core a SQL, MongoDB a BSON, etc.).
- Construís queries dinámicamente.
- Usás el patrón Specification:

```csharp
public class ActivePoliciesSpec
{
    public Expression<Func<Policy, bool>> Predicate =>
        p => p.Active && p.ExpiresAt > DateTime.UtcNow;
}

var spec = new ActivePoliciesSpec();
var policies = await _db.Policies.Where(spec.Predicate).ToListAsync();
// ↑ Se traduce a SQL correctamente
```

**Para forzar evaluación en memoria de forma intencional:**
```csharp
var activos = _db.Policies
    .Where(p => p.Active)            // SQL: WHERE Active = 1
    .AsEnumerable()                   // ← desde acá es LINQ to Objects
    .Where(filtroFuncCompilado)      // se evalúa en memoria
    .ToList();
```

---

## What is the anonymous method in C#? 

An anonymous method is a method without a name. 
It allows you to define a method on the fly without explicitly defining a separate method.

```csharp
// Anonymous method
delegate void PrintDelegate(string message);
PrintDelegate print = delegate (string message)
{
    Console.WriteLine(message);
};

print("Hello, World!"); // Output: Hello, World!
```

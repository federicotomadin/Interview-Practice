# 4. Datos y red

Optimizaciones cuando el cuello de botella está en fetches, datasets grandes o streams en tiempo real.

## TanStack Query (React Query)

Librería de **server state**: cache, sincronización y ciclo de vida de datos del servidor.

La idea central: el estado del servidor no es como el estado de UI — es asíncrono, puede estar stale, y es compartido entre componentes. Manejarlo con `useState` + `useEffect` te obliga a reinventar loading/error/cache/refetch en cada componente.

```tsx
import { useQuery } from '@tanstack/react-query';

function TickerDetail({ symbol }: { symbol: string }) {
  const { data, isLoading, error } = useQuery({
    queryKey: ['ticker', symbol],
    queryFn: () => fetch(`/api/tickers/${symbol}`).then(r => r.json()),
    staleTime: 60_000, // considera el dato fresco por 1 min
  });

  if (isLoading) return <Spinner />;
  if (error) return <ErrorMsg />;
  return <Chart data={data} />;
}
```

### Qué te da gratis

| Feature | Descripción |
|---------|-------------|
| **Cache por queryKey** | Dos componentes piden `['ticker', 'AAPL']` → un solo fetch |
| **Deduplicación** | 10 montajes simultáneos del mismo query = 1 request |
| **Stale-while-revalidate** | Muestra cache al instante, refetchea en background |
| **Refetch on focus** | Al volver a la ventana, actualiza datos stale |
| **Retry con backoff** | Reintentos automáticos en errores de red |
| **Invalidación** | `queryClient.invalidateQueries({ queryKey: ['tickers'] })` |
| **Mutations** | Optimistic updates para UX instantánea |

Ver guía completa en [tanstack-query.md](./tanstack-query.md).

### Analogía .NET

Es como tener `IMemoryCache` + Polly retry + invalidación declarativa, pero del lado del cliente y atado al ciclo de vida de los componentes.

## Paginación vs datasets completos

No traigas 10.000 registros si el usuario ve 50:

- **Paginación clásica** — offset/limit o cursor
- **Infinite scroll** — `useInfiniteQuery` + IntersectionObserver. Ver [infinite-scroll.md](./infinite-scroll.md)

Preferí **cursor-based pagination** sobre offset cuando los datos cambian entre requests (evita duplicados si se insertó una fila).

## WebSockets — batching de updates

Para feeds de mercado en tiempo real: no actualices React en cada tick.

```tsx
// ❌ 100 ticks/segundo = 100 setState/segundo
ws.onmessage = (e) => setPrice(JSON.parse(e.data));

// ✅ Batch cada 150ms
const buffer = useRef<Map<string, number>>(new Map());
ws.onmessage = (e) => {
  const { symbol, price } = JSON.parse(e.data);
  buffer.current.set(symbol, price);
};
setInterval(() => flushBufferToState(), 150);
```

Combiná con selectores granulares en Zustand para que solo re-rendericen las filas cuyo precio cambió.

## Prefetch al navegar

Precargá chunk + datos en paralelo al detectar intención de navegación:

```tsx
import { useQueryClient } from '@tanstack/react-query';

function NavLink({ to, queryKey, queryFn, children }) {
  const queryClient = useQueryClient();
  const loadPage = () => import(`./pages/${to}`);

  return (
    <Link
      to={to}
      onMouseEnter={() => {
        loadPage();
        queryClient.prefetchQuery({ queryKey, queryFn });
      }}
    >
      {children}
    </Link>
  );
}
```

## Checklist

- [ ] ¿Hay fetches duplicados que TanStack Query podría deduplicar?
- [ ] ¿`staleTime` está configurado según la frescura que necesita el dato?
- [ ] ¿Listas grandes usan paginación o infinite scroll?
- [ ] ¿WebSockets batchean updates antes de tocar el estado de React?
- [ ] ¿Infinite scroll + virtualización si acumulás miles de items?

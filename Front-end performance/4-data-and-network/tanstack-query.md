# TanStack Query — Guía práctica

## Setup básico

```tsx
// main.tsx
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 30_000,       // 30s antes de considerar stale
      gcTime: 5 * 60_000,      // 5min en cache después de unmount
      retry: 2,
      refetchOnWindowFocus: true,
    },
  },
});

<QueryClientProvider client={queryClient}>
  <App />
</QueryClientProvider>
```

## useQuery — lectura

```tsx
const { data, isLoading, isFetching, error, refetch } = useQuery({
  queryKey: ['tickers', { sector, sort }],
  queryFn: () => fetchTickers({ sector, sort }),
  staleTime: 60_000,
  enabled: !!sector, // no fetch hasta que sector tenga valor
});
```

### Estados importantes

| Estado | Significado |
|--------|-------------|
| `isLoading` | Primera carga, sin data en cache |
| `isFetching` | Hay un fetch en curso (incluye background refetch) |
| `isStale` | Data en cache pero considerada desactualizada |
| `isError` | El último fetch falló |

## useMutation — escritura

```tsx
const queryClient = useQueryClient();

const mutation = useMutation({
  mutationFn: (newTicker) => api.createTicker(newTicker),
  onSuccess: () => {
    queryClient.invalidateQueries({ queryKey: ['tickers'] });
  },
});

mutation.mutate({ symbol: 'TSLA', name: 'Tesla' });
```

### Optimistic update

```tsx
const mutation = useMutation({
  mutationFn: updatePrice,
  onMutate: async (newPrice) => {
    await queryClient.cancelQueries({ queryKey: ['ticker', symbol] });
    const previous = queryClient.getQueryData(['ticker', symbol]);
    queryClient.setQueryData(['ticker', symbol], newPrice);
    return { previous };
  },
  onError: (_err, _vars, context) => {
    queryClient.setQueryData(['ticker', symbol], context.previous);
  },
  onSettled: () => {
    queryClient.invalidateQueries({ queryKey: ['ticker', symbol] });
  },
});
```

## queryKey — diseño

La `queryKey` es el identificador del cache. Incluí todo lo que afecte el resultado:

```tsx
// ✅ Bueno — distingue por parámetros
['tickers', { page, sector, sort }]
['ticker', symbol]
['user', userId, 'watchlist']

// ❌ Malo — una key para todo
['tickers']
```

## staleTime vs gcTime

| Opción | Qué controla |
|--------|--------------|
| `staleTime` | Cuánto tiempo el dato se considera **fresco** (no refetch automático) |
| `gcTime` (antes `cacheTime`) | Cuánto tiempo permanece en **memoria** después de que ningún componente lo use |

```tsx
// Datos de mercado — refrescar seguido
staleTime: 5_000

// Config de usuario — casi estático
staleTime: 10 * 60_000

// Catálogo de referencia — muy estático
staleTime: Infinity
```

## Invalidación selectiva

```tsx
// Invalida todas las queries de tickers
queryClient.invalidateQueries({ queryKey: ['tickers'] });

// Invalida solo un ticker
queryClient.invalidateQueries({ queryKey: ['ticker', 'AAPL'] });

// Refetch inmediato
queryClient.invalidateQueries({ queryKey: ['tickers'], refetchType: 'active' });
```

## DevTools

```bash
npm i @tanstack/react-query-devtools
```

```tsx
import { ReactQueryDevtools } from '@tanstack/react-query-devtools';

<ReactQueryDevtools initialIsOpen={false} />
```

Útil para ver qué queries están en cache, stale, fetching.

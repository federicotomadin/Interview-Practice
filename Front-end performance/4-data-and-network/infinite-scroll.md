# Infinite scroll

Implementación con `useInfiniteQuery` + `IntersectionObserver`.

## Backend — paginación por cursor

Cada respuesta devuelve `{ items, nextCursor }`. Cursor es más robusto que offset cuando los datos cambian entre requests (nadie ve duplicados si se insertó una fila).

```json
{
  "items": [{ "symbol": "AAPL", "price": 175.2 }],
  "nextCursor": "eyJpZCI6MTIzfQ=="
}
```

## Frontend completo

```tsx
import { useInfiniteQuery } from '@tanstack/react-query';
import { useEffect, useRef } from 'react';

function TickerFeed() {
  const { data, fetchNextPage, hasNextPage, isFetchingNextPage, isLoading } =
    useInfiniteQuery({
      queryKey: ['tickers'],
      queryFn: ({ pageParam }) =>
        fetch(`/api/tickers?cursor=${pageParam}&limit=50`).then(r => r.json()),
      initialPageParam: 0,
      getNextPageParam: (lastPage) => lastPage.nextCursor ?? undefined,
    });

  const sentinelRef = useRef<HTMLDivElement>(null);

  useEffect(() => {
    const observer = new IntersectionObserver(([entry]) => {
      if (entry.isIntersecting && hasNextPage && !isFetchingNextPage) {
        fetchNextPage();
      }
    });
    if (sentinelRef.current) observer.observe(sentinelRef.current);
    return () => observer.disconnect();
  }, [hasNextPage, isFetchingNextPage, fetchNextPage]);

  if (isLoading) return <Spinner />;

  return (
    <div>
      {data?.pages.flatMap(page =>
        page.items.map(t => <Row key={t.symbol} ticker={t} />)
      )}
      <div ref={sentinelRef} style={{ height: 1 }} />
      {isFetchingNextPage && <Spinner />}
    </div>
  );
}
```

## Cómo funciona

| Pieza | Rol |
|-------|-----|
| `initialPageParam` | Valor del cursor para la primera página |
| `getNextPageParam` | Devuelve el cursor de la siguiente página, o `undefined` si no hay más |
| `sentinelRef` | Div invisible al final de la lista |
| `IntersectionObserver` | Dispara `fetchNextPage()` cuando el sentinel entra al viewport |
| `data.pages` | Array de páneas; se aplana con `flatMap` para renderizar |

## Por qué IntersectionObserver y no onScroll

- No necesitás throttle — el browser lo maneja nativamente
- Solo se dispara cuando el sentinel es visible, no en cada pixel de scroll
- Mejor performance que calcular `scrollTop + clientHeight >= scrollHeight`

## Infinite scroll + virtualización

Si acumulás miles de items (200 páginas × 50 items = 10.000 filas), combiná con `@tanstack/react-virtual`:

```tsx
import { useVirtualizer } from '@tanstack/react-virtual';

function VirtualizedFeed() {
  const { data, fetchNextPage, hasNextPage, isFetchingNextPage } =
    useInfiniteQuery({ /* ... */ });

  const allItems = data?.pages.flatMap(p => p.items) ?? [];
  const parentRef = useRef<HTMLDivElement>(null);

  const virtualizer = useVirtualizer({
    count: allItems.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 48,
    overscan: 10,
  });

  // Cargar más cuando el último item virtualizado se acerca al final
  useEffect(() => {
    const lastItem = virtualizer.getVirtualItems().at(-1);
    if (!lastItem) return;
    if (lastItem.index >= allItems.length - 5 && hasNextPage && !isFetchingNextPage) {
      fetchNextPage();
    }
  }, [virtualizer.getVirtualItems(), allItems.length, hasNextPage, isFetchingNextPage]);

  return (
    <div ref={parentRef} style={{ height: 600, overflow: 'auto' }}>
      <div style={{ height: virtualizer.getTotalSize(), position: 'relative' }}>
        {virtualizer.getVirtualItems().map(item => (
          <div
            key={item.key}
            style={{
              position: 'absolute',
              top: 0,
              transform: `translateY(${item.start}px)`,
              height: item.size,
            }}
          >
            <Row ticker={allItems[item.index]} />
          </div>
        ))}
      </div>
    </div>
  );
}
```

El DOM se mantiene chico (~30 nodos) aunque hayas cargado miles de items en memoria.

## Errores comunes

| Error | Fix |
|-------|-----|
| Keys duplicadas al paginar | Usar cursor, no offset; keys únicas por item |
| Fetch en loop infinito | Verificar `hasNextPage` y `isFetchingNextPage` antes de `fetchNextPage` |
| Scroll jump al cargar | Mantener scroll position o usar virtualizer |
| Memoria crece sin límite | Virtualizar + considerar "window" de páginas en cache |

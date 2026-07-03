# Virtualización de listas

## Qué es

Renderizar **solo lo que se ve**. Si tenés 5000 filas pero el viewport muestra 25, sin virtualización el DOM tiene 5000 nodos — la mayoría invisibles pero consumiendo memoria, layout y re-renders.

La virtualización (windowing) hace esto:

1. Un contenedor con la **altura total simulada** (5000 filas × 40px = 200.000px) para que el scrollbar sea correcto.
2. Solo se montan las ~25–30 filas visibles + un pequeño buffer.
3. Al scrollear, se calcula qué índices entran al viewport y se **reciclan** los nodos: la fila que sale por arriba se reposiciona abajo con datos nuevos.

El DOM pasa de 5000 nodos a ~30, siempre, sin importar el tamaño del dataset.

## Con react-window

```tsx
import { FixedSizeList } from 'react-window';

function TickerList({ tickers }: { tickers: Ticker[] }) {
  return (
    <FixedSizeList
      height={600}
      itemCount={tickers.length}
      itemSize={40}
      width="100%"
    >
      {({ index, style }) => (
        <div style={style}>{tickers[index].symbol}</div>
      )}
    </FixedSizeList>
  );
}
```

## Con @tanstack/react-virtual

Mejor integración con infinite scroll y filas de altura variable:

```tsx
import { useVirtualizer } from '@tanstack/react-virtual';

function VirtualTickerList({ tickers }: { tickers: Ticker[] }) {
  const parentRef = useRef<HTMLDivElement>(null);

  const virtualizer = useVirtualizer({
    count: tickers.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 40,
    overscan: 5,
  });

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
            {tickers[item.index].symbol}
          </div>
        ))}
      </div>
    </div>
  );
}
```

## Cuándo virtualizar

| Escenario | ¿Virtualizar? |
|-----------|---------------|
| < 100 filas simples | Probablemente no |
| 500+ filas o celdas complejas (charts inline) | Sí |
| Infinite scroll que acumula miles de items | Sí — combinar con `useInfiniteQuery` |

## Librerías

- [`react-window`](https://github.com/bvaughn/react-window) — simple, filas fijas o variables
- [`@tanstack/react-virtual`](https://tanstack.com/virtual) — headless, flexible, buen combo con TanStack Query

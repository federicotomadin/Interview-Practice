# 2. Runtime — Re-renders

La causa #1 de lentitud en React: componentes que se re-renderizan sin necesidad.

## Técnicas principales

### React.memo

Memoizá componentes **puros** que reciben las mismas props frecuentemente — pero **solo donde el Profiler muestra que duele**.

```tsx
const TickerRow = React.memo(function TickerRow({ symbol, price }: Props) {
  return (
    <tr>
      <td>{symbol}</td>
      <td>{price}</td>
    </tr>
  );
});
```

Sin evidencia en el Profiler, `memo` solo agrega overhead de comparación de props.

### useMemo / useCallback

Estabilizan referencias (objetos, arrays, funciones) que pasás como props a componentes memoizados.

```tsx
const handleSelect = useCallback((id: string) => {
  setSelected(id);
}, []);

const filtered = useMemo(
  () => tickers.filter(t => t.symbol.includes(query)),
  [tickers, query]
);
```

**Importante:** sin `memo` en el hijo, `useCallback` solo agrega overhead — el hijo re-renderiza igual si el padre re-renderiza.

### Colocación del estado

Bajá el estado lo más cerca posible de donde se usa.

```tsx
// ❌ Un keystroke en el root re-renderiza todo el árbol
function App() {
  const [search, setSearch] = useState('');
  return (
    <>
      <SearchInput value={search} onChange={setSearch} />
      <HeavyDashboard />  {/* re-renderiza en cada tecla */}
    </>
  );
}

// ✅ El estado vive en el componente que lo necesita
function SearchBar() {
  const [search, setSearch] = useState('');
  return <SearchInput value={search} onChange={setSearch} />;
}
```

### Composición con children

Pasar contenido como `children` evita re-renders porque React reutiliza el mismo elemento:

```tsx
function Layout({ children }: { children: React.ReactNode }) {
  const [sidebarOpen, setSidebarOpen] = useState(false);
  return (
    <div>
      <Sidebar open={sidebarOpen} />
      {children}  {/* no re-renderiza cuando sidebarOpen cambia */}
    </div>
  );
}
```

### Selectores granulares en state managers

Con Zustand (o Redux con selectores):

```tsx
// ❌ Cada cambio en el store re-renderiza este componente
const store = useStore();

// ✅ Solo se suscribe al slice que necesita
const price = useStore(s => s.tickers[symbol]?.price);
```

Crítico con datos de WebSocket en tiempo real: si cada tick actualiza un objeto global, todo lo suscripto re-renderiza.

### Listas largas — virtualización

Renderizar 5000 filas es suicidio; renderizás solo las ~30 visibles. Ver [list-virtualization.md](./list-virtualization.md).

### Inputs de alta frecuencia

Para búsqueda, sliders y filtros en tiempo real: debounce, throttle, `useDeferredValue` o `useTransition` (React 18). Ver [debounce-throttle.md](./debounce-throttle.md).

```tsx
// React 18 — marcar actualizaciones como no urgentes
const [query, setQuery] = useState('');
const deferredQuery = useDeferredValue(query);

const filtered = useMemo(
  () => heavyFilter(tickers, deferredQuery),
  [tickers, deferredQuery]
);
```

### WebSockets — batching de updates

Acumulá ticks y actualizá el estado cada 100–250ms en vez de por cada mensaje. El ojo humano no nota la diferencia; React sí.

```tsx
const bufferRef = useRef<Map<string, number>>(new Map());

useEffect(() => {
  const ws = new WebSocket(WS_URL);

  ws.onmessage = (event) => {
    const { symbol, price } = JSON.parse(event.data);
    bufferRef.current.set(symbol, price);
  };

  const interval = setInterval(() => {
    if (bufferRef.current.size === 0) return;
    setTickers(prev => {
      const next = new Map(prev);
      bufferRef.current.forEach((price, symbol) => next.set(symbol, price));
      bufferRef.current.clear();
      return next;
    });
  }, 150);

  return () => { ws.close(); clearInterval(interval); };
}, []);
```

## Orden de prioridad

1. Arquitectura del estado (ubicación, granularidad de suscripciones)
2. Virtualización de listas largas
3. Debounce / deferred updates en inputs
4. `memo` / `useMemo` / `useCallback` — donde el Profiler lo justifique

# Debounce y Throttle

## El problema

Un input de búsqueda dispara `onChange` en cada keystroke. Si con cada tecla hacés un fetch o filtrás 5000 tickers, escribir `"TSLA"` ejecuta ese trabajo 4 veces en ~400ms.

## Debounce — "esperá a que el usuario deje de escribir"

Solo ejecuta cuando pasaron X ms **sin nuevos eventos**.

```tsx
const [query, setQuery] = useState('');
const [debounced, setDebounced] = useState('');

useEffect(() => {
  const t = setTimeout(() => setDebounced(query), 300);
  return () => clearTimeout(t); // cancela si llega otra tecla antes de 300ms
}, [query]);

// el fetch/filtro usa `debounced`, no `query`
useEffect(() => {
  if (!debounced) return;
  fetchResults(debounced);
}, [debounced]);
```

Escribís `"TSLA"` → solo se ejecuta **una vez**, 300ms después de la última tecla.

### Con lodash-es

```tsx
import { debounce } from 'lodash-es';
import { useMemo, useEffect } from 'react';

const debouncedSearch = useMemo(
  () => debounce((q: string) => fetchResults(q), 300),
  []
);

useEffect(() => () => debouncedSearch.cancel(), [debouncedSearch]);
```

## Throttle — "ejecutá como máximo una vez cada X ms"

No espera a que pare; garantiza ejecuciones periódicas. Ideal para scroll, resize, drag de sliders — eventos continuos donde querés feedback durante el movimiento, pero no 200 veces por segundo.

```tsx
import { throttle } from 'lodash-es';

const handleScroll = useMemo(
  () => throttle(() => updateScrollPosition(), 100),
  []
);
```

## Regla mental

| Técnica | Cuándo usarla | Ejemplo |
|---------|---------------|---------|
| **Debounce** | Me importa el valor **final** | Búsqueda, autocompletado, validación de form |
| **Throttle** | Me importa el **progreso** | Scroll infinito manual, resize, slider |

## Alternativas nativas en React 18

```tsx
// useDeferredValue — React decide cuándo actualizar lo no urgente
const deferredQuery = useDeferredValue(query);

// useTransition — marcar setState como transición de baja prioridad
const [isPending, startTransition] = useTransition();

const handleChange = (value: string) => {
  setQuery(value); // urgente: el input responde al instante
  startTransition(() => {
    setFiltered(heavyFilter(tickers, value)); // no urgente
  });
};
```

No reemplazan debounce para APIs (evitar requests), pero sí para filtrado pesado en cliente.

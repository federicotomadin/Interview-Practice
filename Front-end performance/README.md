# Front-end Performance — React

Guía de optimización frontend para aplicaciones React: medir primero, diagnosticar, y recién ahí optimizar.

> **Regla de oro:** `memo` / `useMemo` / `useCallback` son el **último recurso**, no el primero. Primero arreglá la arquitectura del estado (dónde vive, quién se suscribe); la memoización parchea síntomas.

## Índice

| # | Sección | Cuándo aplicarla |
|---|---------|------------------|
| 1 | [Medir primero](./1-measure-first/README.md) | Siempre — antes de tocar código |
| 2 | [Runtime — Re-renders](./2-runtime-re-renders/README.md) | La app se siente lenta al interactuar |
| 3 | [Carga inicial — Bundle](./3-initial-load-bundle/README.md) | Tarda en aparecer / LCP malo |
| 4 | [Datos y red](./4-data-and-network/README.md) | Fetches redundantes, datasets grandes, WebSockets |

## Mapa mental

```
React Performance
│
├── 1. MEDIR PRIMERO
│   ├── React DevTools Profiler (re-renders, "why did this render?")
│   ├── Chrome DevTools → Performance (long tasks, scripting vs rendering)
│   ├── Lighthouse / Web Vitals (LCP, INP, CLS)
│   └── Producción: web-vitals library (RUM)
│
├── 2. RUNTIME — Re-renders
│   ├── React.memo (solo donde el Profiler duele)
│   ├── useMemo / useCallback (estabilizar refs para hijos memoizados)
│   ├── Estado cerca de donde se usa (no en el root)
│   ├── Composición con children
│   ├── Selectores granulares (Zustand, Redux)
│   ├── Virtualización de listas (react-window, @tanstack/react-virtual)
│   └── Inputs alta frecuencia → debounce / throttle / useDeferredValue
│
├── 3. CARGA INICIAL — Bundle
│   ├── Code splitting (lazy + Suspense por ruta y por feature)
│   ├── Analizar bundle (rollup-plugin-visualizer)
│   ├── Tree shaking (ESM, evitar barrel files)
│   ├── Vendor chunks (cache de largo plazo)
│   ├── Imágenes (WebP/AVIF, lazy, dimensiones → CLS)
│   └── SSR/SSG (Next.js) si LCP sigue mal
│
└── 4. DATOS Y RED
    ├── TanStack Query (cache, deduplicación, staleTime)
    ├── Paginación / infinite scroll
    └── WebSockets → batching de updates (100–250ms)
```

## Flujo de diagnóstico

```
Síntoma
  └── Profiler / Lighthouse
        ├── ¿Carga lenta?     → Sección 3 (bundle analyzer → code splitting / deps)
        ├── ¿Interacción lenta? → Sección 2 (¿qué re-renderiza y por qué?)
        └── ¿Red / datos?     → Sección 4 (cache, paginación, batching)
```

## Core Web Vitals (referencia rápida)

| Métrica | Qué mide | Bueno | Malo |
|---------|----------|-------|------|
| **LCP** (Largest Contentful Paint) | Tiempo hasta que aparece el elemento principal | < 2.5s | > 4s |
| **INP** (Interaction to Next Paint) | Latencia de respuesta a interacciones | < 200ms | > 500ms |
| **CLS** (Cumulative Layout Shift) | Cuánto "salta" el contenido visualmente | < 0.1 | > 0.25 |

Ver detalle de CLS en [3-initial-load-bundle/core-web-vitals.md](./3-initial-load-bundle/core-web-vitals.md).

## Combo típico en feeds de datos

Infinite scroll + virtualización: cargás páginas con `useInfiniteQuery` pero el DOM se mantiene chico con `@tanstack/react-virtual`. Ver [4-data-and-network/infinite-scroll.md](./4-data-and-network/infinite-scroll.md).

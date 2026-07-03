# 3. Carga inicial — Bundle

## El problema

Cuando un usuario entra a tu SPA, el browser tiene que:

1. Descargar el HTML
2. Descubrir el `<script>`
3. Descargar el bundle de JavaScript
4. Parsearlo y ejecutarlo
5. Recién ahí React monta y pinta algo útil

Todo lo anterior escala con el **tamaño del bundle**. Un bundle de 2MB en un celular de gama media con 4G puede significar 5–8 segundos de pantalla en blanco.

**La clave:** el usuario paga el costo de TODO tu código en la primera visita, aunque solo use el 10%. Alguien que entra al login está descargando también el dashboard, los charts, el módulo de settings y la librería de PDF export.

## Code splitting

Partir el bundle único en chunks que se cargan bajo demanda. El bundler (Vite/Rollup, webpack) detecta los puntos de corte con `import()` dinámico.

### Nivel 1 — por ruta (máximo impacto, empezá acá)

```tsx
import { lazy, Suspense } from 'react';
import { Routes, Route } from 'react-router-dom';

const Dashboard = lazy(() => import('./pages/Dashboard'));
const Screener  = lazy(() => import('./pages/Screener'));
const Settings  = lazy(() => import('./pages/Settings'));

function App() {
  return (
    <Suspense fallback={<PageSkeleton />}>
      <Routes>
        <Route path="/login" element={<Login />} />
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/screener" element={<Screener />} />
        <Route path="/settings" element={<Settings />} />
      </Routes>
    </Suspense>
  );
}
```

- `lazy()` recibe una función que devuelve el `import()` dinámico.
- React no ejecuta ese import hasta que el componente se intenta renderizar por primera vez.
- `Suspense` define qué mostrar mientras el chunk viaja por la red.

### Nivel 2 — por feature pesada

Componentes que arrastran dependencias grandes, aunque estén en una ruta ya cargada:

```tsx
const ChartModal = lazy(() => import('./components/ChartModal'));

function TickerRow({ symbol }: { symbol: string }) {
  const [showChart, setShowChart] = useState(false);
  return (
    <>
      <button onClick={() => setShowChart(true)}>Ver gráfico</button>
      {showChart && (
        <Suspense fallback={<Spinner />}>
          <ChartModal symbol={symbol} />
        </Suspense>
      )}
    </>
  );
}
```

### Preloading — evitar espera al navegar

```tsx
const loadDashboard = () => import('./pages/Dashboard');
const Dashboard = lazy(loadDashboard);

// precarga al hacer hover — cuando clickee, el chunk ya llegó
<Link to="/dashboard" onMouseEnter={loadDashboard}>Dashboard</Link>
```

Combiná con TanStack Query: al hover podés precargar chunk **y** datos en paralelo (`prefetchQuery`).

## Analizar el bundle

Antes de optimizar dependencias, necesitás ver qué contiene el bundle.

### Vite + rollup-plugin-visualizer

```bash
npm i -D rollup-plugin-visualizer
```

```ts
// vite.config.ts
import { visualizer } from 'rollup-plugin-visualizer';

export default defineConfig({
  plugins: [
    react(),
    visualizer({ open: true, gzipSize: true }),
  ],
});
```

Al hacer `npm run build` se abre un treemap: rectángulos proporcionales al peso de cada módulo.

### Hallazgos típicos

| Dependencia | Problema | Alternativa |
|-------------|----------|-------------|
| moment.js (~290kb con locales) | Peso + locales enteros | date-fns o dayjs (~7kb) |
| lodash completo (~70kb) | Importás 3 funciones, traés todo | `lodash-es` con named imports |
| Librerías de charts/UI | Importadas para un solo componente | lazy load al punto de uso |
| Polyfills duplicados | Dos versiones de la misma dep | `npm ls <paquete>` |
| react-icons | Puede arrastrar sets completos | Verificar tree shaking del bundler |

## Tree shaking

El bundler elimina exports que nadie importa — pero solo si puede probar estáticamente que no se usan. Requiere módulos **ES** (`import`/`export`), no CommonJS (`require`).

```ts
// ❌ Mata el tree shaking
import _ from 'lodash';
_.debounce(fn, 300);

// ✅ Named import de paquete ESM
import { debounce } from 'lodash-es';

// ✅ Import directo del módulo
import debounce from 'lodash/debounce';
```

### Qué rompe tree shaking silenciosamente

- Dependencias publicadas solo en CommonJS
- **Side effects:** módulos que ejecutan código al importarse (registran globals, inyectan CSS). Las librerías bien hechas declaran `"sideEffects": false` en `package.json`.
- **Barrel files** (`index.ts` que re-exporta todo): importar una cosa del barrel puede traer el archivo entero con dependencias transitivas.

## Vendor chunks y caching

Partí el código por frecuencia de cambio. Tu código cambia con cada deploy; React cambia cada varios meses.

```ts
// vite.config.ts
build: {
  rollupOptions: {
    output: {
      manualChunks: {
        'vendor-react': ['react', 'react-dom', 'react-router-dom'],
        'vendor-query': ['@tanstack/react-query'],
      },
    },
  },
},
```

Los archivos salen con hash en el nombre (`vendor-react-a1b2c3.js`). Si el contenido no cambió, el hash no cambia, y el browser sirve el chunk desde cache. En deploys sucesivos el usuario solo re-descarga tu código de aplicación (~100kb), no los 140kb de React.

## Imágenes

Ver [core-web-vitals.md](./core-web-vitals.md) para CLS e imágenes.

```html
<img
  src="/chart-preview.webp"
  width="800"
  height="450"
  loading="lazy"
  decoding="async"
  alt="Preview"
/>
```

- **WebP/AVIF:** 30–60% menos que JPEG/PNG equivalentes
- **`loading="lazy"`:** imágenes below the fold — nunca en la imagen LCP
- **`srcset`:** resoluciones según dispositivo
- **Preload LCP:** `<link rel="preload" as="image" href="...">`

## SSR / SSG

Si el LCP sigue mal después de optimizar bundle e imágenes: SSR/SSG con Next.js, o al menos preload de recursos críticos.

## Proceso completo (orden de impacto)

1. **visualizer** → identificá el 20% que pesa el 80%
2. **Split por ruta** con `lazy` + `Suspense` (una tarde, impacto enorme)
3. **Reemplazá** las 2–3 dependencias más pesadas del treemap
4. **Split de features** pesadas (charts, editores, PDF) al punto de uso
5. **Vendor chunks** para cache de largo plazo
6. **Imágenes:** dimensiones + formatos + lazy
7. **Lighthouse** antes/después — cerrás el ciclo midiendo

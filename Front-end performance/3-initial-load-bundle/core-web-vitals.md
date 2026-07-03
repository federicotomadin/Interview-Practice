# Core Web Vitals — LCP, INP, CLS

## LCP (Largest Contentful Paint)

Cuánto tarda en aparecer el **elemento principal** de la página (hero image, título grande, bloque de contenido).

| Score | Valor |
|-------|-------|
| Bueno | < 2.5s |
| Necesita mejora | 2.5s – 4s |
| Malo | > 4s |

**Fixes:** code splitting, imágenes optimizadas, preload del recurso LCP, SSR/SSG.

## INP (Interaction to Next Paint)

Cuánto tarda la página en **responder a interacciones** (click, tap, tecla). Reemplazó a FID como métrica de interactividad.

| Score | Valor |
|-------|-------|
| Bueno | < 200ms |
| Necesita mejora | 200ms – 500ms |
| Malo | > 500ms |

**Fixes:** reducir re-renders, debounce/throttle, evitar long tasks en el main thread, `useTransition` para updates no urgentes.

## CLS (Cumulative Layout Shift)

Mide cuánto **"salta"** el contenido visualmente mientras carga la página.

### El caso típico

Estás por tocar un botón → carga una imagen o un banner arriba → todo se desplaza → tocás otra cosa. Cada movimiento inesperado de layout suma al score.

| Score | Valor |
|-------|-------|
| Bueno | < 0.1 |
| Necesita mejora | 0.1 – 0.25 |
| Malo | > 0.25 |

### Causas comunes y fixes

| Causa | Fix |
|-------|-----|
| Imágenes sin dimensiones | Siempre `width`/`height` explícitos — el browser reserva espacio antes de cargar |
| Contenido inyectado dinámicamente arriba del fold | Reservá espacio con skeletons de altura fija |
| Fuentes web que cambian métricas al cargar (FOUT) | `font-display: optional` o preload de la fuente |
| Ads / banners que aparecen tarde | Reservar slot con min-height |
| Iframes embebidos sin tamaño | Definir dimensiones fijas |

### Ejemplo — imagen sin CLS

```html
<!-- ❌ Sin dimensiones: el layout salta cuando carga -->
<img src="/hero.jpg" alt="Hero" />

<!-- ✅ El browser reserva 800×450px desde el primer paint -->
<img
  src="/hero.webp"
  width="800"
  height="450"
  alt="Hero"
/>
```

### Ejemplo — skeleton con altura fija

```tsx
function FeedSkeleton() {
  return (
    <div style={{ minHeight: 600 }}>
      {Array.from({ length: 10 }).map((_, i) => (
        <div key={i} style={{ height: 48, marginBottom: 8, background: '#eee' }} />
      ))}
    </div>
  );
}
```

## Medición

- **Lab:** Lighthouse en Chrome DevTools
- **Campo:** librería `web-vitals` en producción
- **Search Console:** reporte de Core Web Vitals por URL real

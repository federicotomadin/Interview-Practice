# 1. Medir primero

Antes de tocar código, necesitás datos. Optimizar sin medir es la trampa clásica.

Esto divide el problema en dos familias:

- **Carga inicial lenta** → bundle, imágenes, SSR
- **Interacciones lentas** → re-renders, estado mal ubicado, listas sin virtualizar

## Herramientas

### React DevTools Profiler

- Grabá una interacción y mirá qué componentes re-renderizan, cuántas veces y por qué.
- Activá **"Record why each component rendered"** — es oro para diagnosticar.
- Útil cuando la app se siente lenta al escribir, scrollear o hacer click.

### Chrome DevTools → Performance tab

- Long tasks (> 50ms bloquean el main thread).
- Layout thrashing (lecturas/escrituras DOM alternadas).
- Tiempo de **scripting** vs **rendering** vs **painting**.
- Útil para ver si el cuello es JS, layout o paint.

### Lighthouse / Web Vitals

| Métrica | Problema que indica |
|---------|---------------------|
| **LCP** | Carga inicial — bundle grande, imagen hero lenta, sin SSR |
| **INP** | Interactividad — re-renders, handlers pesados, main thread bloqueado |
| **CLS** | Layout inestable — imágenes sin dimensiones, contenido inyectado arriba |

Corré Lighthouse en modo incógnito, throttling de red/CPU activado, para simular usuarios reales.

### Producción — Real User Monitoring (RUM)

La librería [`web-vitals`](https://github.com/GoogleChrome/web-vitals) envía métricas reales de usuarios:

```tsx
import { onLCP, onINP, onCLS } from 'web-vitals';

function sendToAnalytics({ name, value, id }) {
  // tu endpoint de analytics
  fetch('/api/vitals', {
    method: 'POST',
    body: JSON.stringify({ name, value, id }),
  });
}

onLCP(sendToAnalytics);
onINP(sendToAnalytics);
onCLS(sendToAnalytics);
```

Lab (Lighthouse) y campo (RUM) suelen divergir — confiá en RUM para priorizar.

## Checklist antes de optimizar

- [ ] ¿Reproduciste el problema con Profiler o Performance tab?
- [ ] ¿Sabés si es carga inicial o interacción?
- [ ] ¿Tenés un baseline (LCP, INP, CLS) antes del cambio?
- [ ] ¿Vas a medir de nuevo después del cambio?

## Siguiente paso según diagnóstico

| Síntoma | Ir a |
|---------|------|
| Pantalla en blanco larga al abrir | [3-initial-load-bundle](../3-initial-load-bundle/README.md) |
| UI congelada al escribir / scrollear | [2-runtime-re-renders](../2-runtime-re-renders/README.md) |
| Muchos requests, datos duplicados, listas enormes | [4-data-and-network](../4-data-and-network/README.md) |

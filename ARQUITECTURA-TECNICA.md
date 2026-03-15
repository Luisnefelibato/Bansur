# Arquitectura técnica — Página sencilla Remesas Bansur

Documento de referencia para el diseño e implementación técnica de la **página web sencilla e informativa** de Remesas Bansur. Incluye exhibición de servicios, calculadora/cotizaciones en pantalla y opcional formulario de contacto. **No hay backend para procesar envíos de remesas.**

---

## 1. Visión general y alcance

### 1.1 Objetivos técnicos

- **Página corporativa e informativa**: Presentación de servicios (remesas, transferencias, pagos, recargas, depósitos, seguros), Sobre nosotros, Beneficios, Cómo funciona, Confianza. Sin transacciones de envío; solo exhibición y generación de leads.
- **Calculadora o cotización**: Se puede incluir un bloque/calculadora que muestre tipo de cambio y monto a recibir (datos estáticos o API de solo lectura). Solo visualización; **no se realizan envíos** desde esta página.
- **Sin backend de envíos**: No existe backend para registrar usuarios, procesar remesas ni integrar pasarelas de pago para enviar dinero. Opcional: backend mínimo para formulario de contacto y/o API de solo lectura para datos de la calculadora.
- **Requisitos no funcionales**: Rendimiento (LCP < 2.5s, FID < 100ms, CLS < 0.1), accesibilidad WCAG 2.1 AA, SEO, seguridad, disponibilidad ≥ 99.5%.

### 1.2 Usuarios y dispositivos

- Usuarios finales: emisores y receptores de remesas, consulta de servicios. Principalmente móvil y desktop.
- Navegadores objetivo: Chrome, Firefox, Safari, Edge (últimas 2 versiones). Soporte razonable en navegadores legacy (ej. IE11) solo si es requisito explícito.
- Resoluciones: desde 320px (móvil) hasta 1920px+ (desktop). Diseño responsive y touch-friendly.

---

## 2. Stack tecnológico

### 2.1 Frontend

| Capa | Tecnología recomendada | Alternativas | Notas |
|------|------------------------|-------------|-------|
| **Marcado** | HTML5 semántico | — | `<header>`, `<main>`, `<section>`, `<article>`, landmarks ARIA |
| **Estilos** | CSS3 (variables, Grid, Flexbox) | Sass/SCSS si se usa build | Critical CSS inline para above-the-fold; resto en archivos bloqueados por media |
| **Scripts** | Vanilla JS (ES2020+) o React/Vue | Next.js si se requiere SSR/SEO avanzado | Evitar dependencias pesadas en la landing; code splitting por ruta en SPA |
| **Fuentes** | Google Fonts (Cormorant Garamond, Outfit) o self-hosted | Variable fonts para reducir requests | `font-display: swap`; preload para fuentes críticas |
| **Imágenes** | WebP + AVIF con fallback JPEG/PNG | — | Responsive images (`srcset`, `sizes`); lazy loading nativo `loading="lazy"` |
| **Iconos** | SVG inline o sprite | Icon font solo si ya existe | Accesibilidad: `aria-hidden="true"` o texto alternativo según contexto |
| **Build** | Vite o Parcel | Webpack, Rollup | Minificación, tree-shaking, hashing de assets, generación de service worker (PWA opcional) |

### 2.2 Backend (mínimo, solo si aplica)

- **No hay backend para envíos de remesas.** No se implementan: registro de usuarios, procesamiento de órdenes de remesa, ni pasarela de pago para cobrar envíos.
- **Opcional:** (1) **Formulario de contacto**: envío vía Netlify Forms, Formspree o API mínima que solo reciba el mensaje. (2) **Calculadora/cotización**: si se usan datos en vivo, una API o fuente de **solo lectura** (ej. tipo de cambio del día) para mostrar en la calculadora; sin lógica de transacciones.

### 2.3 Integraciones externas

- **Calculadora/cotización**: Opcionalmente, API o JSON de solo lectura para tipo de cambio o tarifas (solo para mostrar en pantalla; no para ejecutar envíos).
- **Formulario de contacto**: Netlify Forms, Formspree o API mínima para recibir mensajes.
- **Analytics**: Google Analytics 4 (o similar) con consentimiento (cookie banner, CMP). Eventos: scroll, clics en CTAs, uso de calculadora, envío de contacto.
- **Soporte**: Chat widget o enlace a canal de contacto si se desea. No se integra con proveedores de remesas para procesar envíos ni con pasarelas de pago.

### 2.4 Hosting e infraestructura

| Elemento | Recomendación | Detalle |
|----------|---------------|---------|
| **Hosting estático** | CDN + origen (Netlify, Vercel, Cloudflare Pages, AWS S3+CloudFront) | SSL/TLS 1.3, HTTP/2, compresión Brotli/gzip |
| **API/Backend** | Solo si se usa (formulario o API lectura); VPS o PaaS mínimo | No requerido para la página sencilla; opcional para contacto o datos calculadora |
| **DNS** | Proveedor con DNSSEC (Cloudflare, AWS Route 53) | TTL bajo para críticos; CAA para certificados |
| **Dominio** | Registro en proveedor de confianza | Renovación automática; bloqueo de transferencia |
| **Entornos** | Producción, preproducción (staging), desarrollo | Variables de entorno por entorno; sin secretos en repo |

---

## 3. Estructura de la aplicación (frontend)

### 3.1 Mapa de páginas y rutas

```
/                     → Landing (Hero, Servicios, Sobre nosotros, Cómo funciona, Confianza, CTA)
/cotizar o #calculadora → Calculadora o cotización (solo visualización; sin envío)
/sobre-nosotros       → [Opcional] Página ampliada Sobre nosotros
/servicios            → [Opcional] Listado detallado de servicios con subpáginas
/contacto             → Formulario de contacto y/o datos de agencias
/aviso-legal          → Aviso legal, condiciones de uso
/politica-privacidad  → Política de privacidad y cookies
/accesibilidad        → Declaración de accesibilidad

No se incluyen: /registro, /login, /envio, /mis-envios, /cuenta (no hay backend de envíos).
```

### 3.2 Componentes reutilizables

- **Header/Nav**: Logo, menú principal, CTA, menú móvil (hamburguesa). Sticky con cambio de estilo al scroll.
- **Footer**: Logo, enlaces legales, contacto, redes, tagline.
- **Sección genérica**: Contenedor con título, subtítulo y bloque de contenido (grid o lista). Parallax opcional.
- **Tarjeta de servicio**: Icono, título, descripción. Variantes: estándar, destacada (bento).
- **Tarjeta de beneficio**: Icono, título, texto. Mismo estilo que en propuesta actual.
- **Bloque CTA**: Fondo destacado (gradiente), título, texto, botón. Reutilizable en hero y sección final.
- **Formulario de contacto**: Campos (nombre, email, asunto, mensaje), validación cliente, envío vía API o formspree/netlify forms.
- **Aviso de cookies / consentimiento**: Banner fijo, enlace a política, botón aceptar/rechazar. Persistencia en `localStorage`.

### 3.3 Design system (tokens)

- **Colores**: `--bg-dark`, `--bg-card`, `--gold`, `--gold-light`, `--gold-dark`, `--teal`, `--teal-light`, `--white`, `--gray`. Modo oscuro como base; variante clara opcional.
- **Tipografía**: Escala modular (ej. 0.875rem – 1.125rem – 1.25rem – 1.5rem – 2rem – 2.5rem – 3rem). `font-family` para títulos (Cormorant Garamond) y cuerpo (Outfit).
- **Espaciado**: Escala 4/8px (4, 8, 12, 16, 24, 32, 48, 64, 96).
- **Bordes**: `border-radius` 8px, 12px, 16px, 20px, 24px según componente.
- **Sombras**: 2–3 niveles (sutil, media, elevada) con color coherente (negro/teal/dorado).
- **Animaciones**: Duración estándar 0.3–0.5s; easing `cubic-bezier(0.22, 1, 0.36, 1)`. Reducir movimiento si `prefers-reduced-motion: reduce`.

---

## 4. Rendimiento

### 4.1 Métricas objetivo (Core Web Vitals)

- **LCP (Largest Contentful Paint)**: < 2.5 s. Priorizar hero (logo, título, CTA) e imágenes above-the-fold.
- **FID / INP (First Input Delay / Interaction to Next Paint)**: < 100 ms. Scripts no bloqueantes; tareas largas divididas o en worker.
- **CLS (Cumulative Layout Shift)**: < 0.1. Dimensiones explícitas en imágenes e iframes; reservar espacio para fuentes (aspect-ratio o min-height).

### 4.2 Estrategias

- **Critical path**: HTML mínimo; critical CSS inline en `<head>`; JS diferido (`defer`/`async`). Resto de CSS/JS cargado de forma asíncrona.
- **Imágenes**: Formato moderno (WebP/AVIF), `srcset`/`sizes`, `loading="lazy"` (excepto hero). Opcional: CDN con redimensionado automático.
- **Fuentes**: `preload` para las 2 familias críticas; `font-display: swap`; subset si es posible.
- **Third-party**: Cargar analytics y widgets después de que la página sea interactiva; si es posible, en `requestIdleCallback` o tras evento de usuario.
- **Caché**: Headers `Cache-Control` largos para assets con hash (1 año); revalidación para HTML. Service Worker opcional para offline básico.

---

## 5. Seguridad

### 5.1 Capa de transporte y contenido

- **HTTPS obligatorio**: TLS 1.2 mínimo; 1.3 recomendado. HSTS header.
- **Headers de seguridad**: `Content-Security-Policy`, `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY` o `SAMEORIGIN`, `Referrer-Policy: strict-origin-when-cross-origin`.
- **Subresource Integrity (SRI)**: Para scripts/estilos cargados desde CDN externos.

### 5.2 Datos y privacidad

- **Formularios**: Validación cliente y servidor. No exponer datos sensibles en URL. En fase operativa, datos de pago fuera del frontend (tokenización o redirección a pasarela).
- **Cookies**: Solo las necesarias; consentimiento según normativa (GDPR/LOPD según jurisdicción). `SameSite`, `Secure`, `HttpOnly` donde aplique.
- **Política de privacidad**: Explicitar recogida, finalidad, base legal, cesiones (proveedores de remesas, email, analytics), derechos y contacto.

### 5.3 Backend (si existe)

- Solo aplica a formulario de contacto o API de solo lectura para la calculadora. Validación y sanitización de entradas; no se almacenan datos de transacciones ni tarjetas. No aplica PCI-DSS ni autenticación de usuarios para envíos.

---

## 6. Accesibilidad (a11y)

- **WCAG 2.1 nivel AA**: Contraste mínimo 4.5:1 texto normal; 3:1 texto grande. Enfoque visible en elementos interactivos.
- **Semántica**: Uso correcto de headings (`h1`–`h6`), listas, `nav`, `main`, `footer`. Landmarks ARIA si hace falta.
- **Formularios**: `<label>` asociados, mensajes de error en vivo, agrupación con `fieldset`/`legend` cuando proceda.
- **Navegación por teclado**: Orden de tabulación lógico; sin trampas de foco; skip link “Saltar al contenido”.
- **Medios**: Texto alternativo en imágenes; transcripción o subtítulos en vídeo/audio si se añaden.
- **Reducción de movimiento**: Respetar `prefers-reduced-motion: reduce` (desactivar o simplificar animaciones).

---

## 7. SEO

- **Meta tags**: `title` y `meta description` por página; `og:title`, `og:description`, `og:image` para redes.
- **Estructura**: Un `h1` por página; jerarquía de headings coherente.
- **URLs**: Amigables, descriptivas, preferiblemente en español para el mercado objetivo.
- **Sitemap**: `sitemap.xml` generado (manual o por build) y enviado a Search Console.
- **Schema.org**: `Organization`, `WebSite`; si hay formulario de contacto o ofertas, `LocalBusiness` o tipos específicos según contenido.
- **Rendimiento**: Los Core Web Vitals forman parte de los criterios de ranking; cumplirlos favorece el SEO.

---

## 8. Monitoreo y operación

- **Disponibilidad**: Ping/health check cada X minutos; alertas si caída o tiempo de respuesta alto.
- **Errores**: Logging de errores en frontend (ej. Sentry) y en backend; revisión y corrección en ciclos cortos.
- **Analytics**: Eventos de negocio (clics en “Ver servicios”, envío de formulario, pasos del flujo de envío) además de páginas vistas.
- **Backups**: Base de datos y archivos críticos con frecuencia definida; pruebas de restauración periódicas.
- **CI/CD**: Build y tests automáticos en cada commit; despliegue a staging; despliegue a producción tras revisión o tag. Variables y secretos en gestor de secretos (no en código).

---

## 9. Resumen de entregables técnicos

| Entregable | Descripción |
|------------|-------------|
| **Repositorio** | Código fuente versionado (Git); README con instrucciones de instalación y despliegue |
| **Documentación** | Este documento; README por proyecto; comentarios en código para lógica no obvia |
| **Entornos** | Staging y producción configurados; URLs y credenciales documentados de forma segura |
| **Tests** | Tests unitarios y E2E para flujos críticos (formularios, navegación, en fase 2: login y envío) |
| **Design system** | Tokens (colores, tipografía, espaciado) y componentes documentados para consistencia |

---

*Documento de arquitectura técnica — Remesas Bansur. Actualizable según decisiones de stack y alcance de fases posteriores.*

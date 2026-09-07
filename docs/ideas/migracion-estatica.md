# Migración a HTML/CSS estático de iepsicoterapia.org

## Problem Statement
¿Cómo sustituir el WordPress actual por una versión estática, idéntica en diseño y contenido, sin perder URLs/SEO ni depender de un formulario que ya no puede procesarse en estático, dado que ya no necesitas editar desde el panel de WP?

## Recommended Direction
Combinar **auditoría de URLs primero** con una **exportación estática simple (Freeze & Serve)**, manteniendo el mismo hosting compartido y el mismo dominio.

La web es puramente informativa (sin reservas de citas, sin comentarios, sin buscador), así que no hay lógica de servidor real que preservar más allá del formulario de contacto — y ese se elimina, dejando enlaces `mailto:`/`tel:` directos. Esto convierte la migración en un cambio de bajo riesgo: se congela el sitio tal cual se ve hoy y se sirve como ficheros planos, eliminando toda la superficie de ataque de PHP/MySQL/plugins sin tocar el hosting, el dominio ni el correo (que vive en la misma cuenta de hosting).

La diferenciación frente a seguir con WordPress no es "más barato" o "más rápido" — es estructural: se pasa de "código ejecutándose en el servidor + plugins desactualizables" a "cero backend", que es una categoría de riesgo distinta.

## Key Assumptions to Validate
- [ ] Las URLs actuales se reproducen igual en el export — comparar el sitemap.xml de WP contra la estructura de carpetas que genera `wget`
- [ ] Scripts de terceros (Analytics u otros) siguen funcionando sin backend — probar en la copia local antes de publicar
- [ ] Tras quitar el formulario de la página de contacto exportada, los enlaces `mailto:`/`tel:` quedan visibles y funcionales — revisar a mano esa página en local

## MVP Scope
**Dentro:** exportar el sitio actual tal cual (mismo diseño, mismo contenido, mismas URLs) con `wget`, quitar el formulario de contacto del HTML descargado, servir el resultado desde el mismo hosting, con redirects si alguna URL no coincide.

**Fuera de esta pasada:** rediseño, limpieza/minificación de código, cambio de proveedor de hosting, cualquier generador estático con partials/plantillas.

## Not Doing (and Why)
- **Migrar a un generador estático con partials (Eleventy/Hugo)** — resolvería la duplicación de cabecera/footer si algún día cambia un dato en todas las páginas, pero añade una herramienta nueva justo cuando el objetivo es *menos* complejidad. Revisitable en el futuro si editar muchos archivos a mano por un cambio puntual se vuelve doloroso.
- **Cambiar de hosting (Cloudflare Pages/Netlify)** — eliminaría PHP/MySQL del todo y sería gratis, pero el hosting actual también sirve el correo del dominio; cambiar de proveedor arriesga esa parte sin necesidad, y no resuelve ningún dolor adicional ahora mismo.
- **Minificar/limpiar el HTML generado por WP** — mejoraría algo el rendimiento, pero mezclar "cambiar de plataforma" con "optimizar rendimiento" en el mismo paso complica verificar que nada se rompió durante la migración.
- **Mantener un backup adicional** — ya existe una copia de seguridad completa de la web actual, no hace falta duplicar ese paso.

## Plan de ejecución

1. ✅ **Auditar URLs actuales** — hecho. 18 URLs de contenido + home identificadas vía API REST de WP (el `wp-sitemap.xml` está roto). Detalle en `docs/ideas/audit-urls.md`. De paso se encontró un enlace roto en producción (`/foro-profesional-de-iep` → 404) — **decisión pospuesta por el usuario**, se arregla más adelante.
2. ✅ **Exportar con `wget --mirror --convert-links --page-requisites`** — hecho. 12 MB en `static-export/iepsicoterapia.org/`. Se completaron a mano 2 páginas que el crawl no descubrió por no estar enlazadas (`/politica-de-cookies/`, `/jornada-inaugural/`), y se limpiaron 15 ficheros `?p=NN` (shortlinks de WP sin valor).
3. ✅ **Editar la página de contacto** (quitar el formulario) — hecho, ver detalle en "Cambios de contenido aplicados tras el export" más abajo. Ya existe un `mailto:masterpsicoterapiauah@gmail.com` en la página (fuera de la sección eliminada); no se ha encontrado ningún `tel:` — pendiente decidir si se añade un teléfono de contacto visible ahora que no queda formulario.
4. ✅ **Validar en local** (`python -m http.server`) — hecho, con una corrección importante encontrada al revisar a ojo (ver abajo). Tras corregirla: las 18 páginas + home devuelven 200, 0 enlaces internos rotos (15 rutas únicas comprobadas), 0 recursos locales rotos, favicon y Google Fonts externos correctos.

   **Bug encontrado y corregido:** el menú principal de WordPress usa "shortlinks" (`/?p=111`) en vez de las URLs bonitas (`/contacto/`) para casi todos sus enlaces — algo que ya pasaba en el WordPress en vivo, pero que WP disimula redirigiendo automáticamente `?p=111` → `/contacto/`. Un servidor estático no hace esa redirección. Al exportar con `wget`, esos shortlinks se guardaron como ficheros con un `?` literal en el nombre (p. ej. `index.html?p=111.html`); en la limpieza del paso 2 se borraron pensando que eran duplicados sin valor, lo que rompió el menú en las 16 páginas que lo incluyen. Corregido reescribiendo esos 14 enlaces a su ruta bonita en formato absoluto (`/contacto/` en vez de relativo), usando el mapeo id→slug de la API REST:

   | id  | slug                                   |
   |-----|-----------------------------------------|
   | 19  | por-que-iep                             |
   | 26  | quienes-somos                           |
   | 33  | equipo-de-direccion                     |
   | 44  | formacion                               |
   | 51  | master-en-psicoterapia-uah-iep          |
   | 60  | talleres-y-cursos                       |
   | 68  | supervision                             |
   | 75  | metodologia                             |
   | 82  | competencias-a-conseguir-por-los-alumnos|
   | 89  | a-quien-nos-dirigimos                   |
   | 96  | acreditaciones                          |
   | 103 | area-clinica                            |
   | 111 | contacto                                |
   | 216 | terminos-y-condiciones-de-uso           |
   | 517 | blog-opiniones-de-psicoterapeutas       |

   **Si se vuelve a exportar con `wget` en el futuro** (p. ej. tras quitar el formulario en el WP en vivo), este mismo problema reaparecerá — hay que repetir esta sustitución (`index.html%3Fp=<id>.html` → `/<slug>/`) antes de dar por bueno el export. No volver a borrar los ficheros `index.html?p=*.html` sin antes comprobar si algún menú los usa.
5. ✅ **Comparar URLs generadas contra la auditoría** — coinciden las 18. Los redirects en `.htaccess` solo harán falta si al final se decide cambiar la ruta rota en vez de solo corregirla (ver paso 1).
6. 🔒 **Sustituir `public_html` en producción** — **bloqueado**: requiere acceso SSH/FTP/cPanel al hosting real, que no está disponible en esta sesión. El usuario ha decidido dejar la decisión de despliegue para más adelante.
7. 🔒 **Vigilar Google Search Console** — depende del paso 6, mismo bloqueo.

**Paquete listo para cuando se decida el despliegue:** `iepsicoterapia-static-<fecha>.zip` en la raíz del proyecto, con el contenido validado de `static-export/iepsicoterapia.org/`.

## Cambios de contenido aplicados tras el export (para repetir si se re-exporta)

Estos dos cambios se hicieron a mano sobre el HTML ya descargado, no en el WordPress en vivo. **Si algún día se vuelve a exportar con `wget`, hay que repetirlos:**

1. **Formulario de contacto eliminado** — en `contacto/index.html` se quitó por completo la `<section>` con `data-id="4009fa5f"` (el widget de formulario de Elementor/Contact Form 7, que no puede procesar envíos sin backend). Solo afecta a esa página. Queda el enlace `<link>` a `contact-form-7/includes/css/styles.css` en el `<head>`, ya sin uso — inofensivo (una hoja de estilos que no se aplica a nada), no se ha quitado.
2. **Buscador móvil eliminado en las 18 páginas** — el `<form class="mobile-searchform">` (búsqueda interna de WordPress, con acción `?s=...`) no tiene backend que la resuelva en estático, así que se eliminó de todas las páginas. Nota: en el DOM ya renderizado por el navegador, el plugin del menú móvil ("sidr") clona este formulario y le añade el prefijo de clase `sidr-class-` — por eso puede aparecer como `form.sidr-class-mobile-searchform` al inspeccionar con las DevTools, aunque en el HTML fuente la clase es solo `mobile-searchform`. Al quitar el formulario original ya no hay nada que clonar.

## Pendientes antes de poder desplegar
- Quitar el formulario de contacto del HTML descargado (paso 3).
- Decidir qué hacer con `/foro-profesional-de-iep` (arreglar el enlace o quitarlo del menú).
- Elegir método de despliegue a producción: acceso SSH/FTP para que se haga en una sesión futura, guía paso a paso por cPanel para hacerlo el propio usuario, u otra vía.

## Open Questions
- Ninguna pendiente sobre el contenido — resueltas durante la sesión: no hay banner de cookies, el hosting sirve correo (no se toca al sustituir solo `public_html`), y ya existe backup de la web actual.

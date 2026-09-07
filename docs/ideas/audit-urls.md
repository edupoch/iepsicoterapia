# Auditoría de URLs — iepsicoterapia.org (paso 1 del plan)

Fuente: API REST de WordPress (`/wp-json/wp/v2/pages`, `/posts`, `/categories`), cruzada con los enlaces reales de la portada. El `wp-sitemap.xml` está roto (devuelve HTTP 200 pero con el HTML de la portada en vez de XML — `content-type: text/html`), así que no se ha usado como fuente.

Contexto del servidor: WordPress 5.7.17 sobre PHP 7.2.34 (ambos sin soporte de seguridad desde hace años).

## URLs a preservar en el export estático

- `/` (inicio)
- `/por-que-iep/`
- `/quienes-somos/`
- `/equipo-de-direccion/`
- `/formacion/`
- `/master-en-psicoterapia-uah-iep/`
- `/talleres-y-cursos/`
- `/supervision/`
- `/metodologia/`
- `/competencias-a-conseguir-por-los-alumnos/`
- `/a-quien-nos-dirigimos/`
- `/acreditaciones/`
- `/area-clinica/`
- `/contacto/`
- `/terminos-y-condiciones-de-uso/`
- `/politica-de-cookies/` (legal, enlazada desde el footer, no desde el menú principal)
- `/blog-opiniones-de-psicoterapeutas/`
- `/jornada-inaugural/` (entrada de blog)

18 URLs de contenido real + la home.

## URLs a excluir del export (ruido de WordPress, no contenido real)

- `/pagina-ejemplo/` — "Sample Page" por defecto de WP, no enlazada desde ningún sitio
- `/category/sin-categoria/` — archivo de categoría por defecto, sin valor
- `/feed/`, `/comments/feed/` — RSS, sin sentido en estático
- `/wp-json/*` — API REST, desaparece junto con WP

## Hallazgo a resolver antes del export

- **Enlace roto en producción:** `/foro-profesional-de-iep` (sin barra final) devuelve **404** ya en la web actual. Decidir si se quita del menú o se corrige antes de congelar el sitio — si no, el estático heredará el enlace muerto.

## Uso de esta lista

En el paso 5 del plan ("comparar URLs generadas contra el sitemap original"), esta lista de 18 URLs + home es la referencia contra la que comprobar lo que produzca `wget --mirror`. Cualquier URL de esta lista que no aparezca en el export, o que aparezca con una ruta distinta, necesita un redirect en `.htaccess`.

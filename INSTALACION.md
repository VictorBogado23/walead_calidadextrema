# WaLead Widget — Guía de instalación

Widget flotante de WhatsApp que **captura el email y el WhatsApp del visitante** (con selector
de país y formato internacional), los **publica en el `dataLayer`** y recién entonces abre WhatsApp con el
mensaje ya escrito. Se instala con **una sola línea** y se personaliza con atributos.

El widget **no guarda datos por su cuenta**: guardar el lead (Google Sheets, CRM) y medir
conversiones se arma en Google Tag Manager a partir del evento `walead_submit`.

Pensado para que lo **comercialices con tus clientes**: alojas un único `widget.js` y a cada
cliente le das su línea de instalación con su configuración.

---

## Archivos del paquete

| Archivo | Para qué sirve |
|---|---|
| `widget.min.js` | El widget, reducido. Lo alojas **tú** una sola vez (tu hosting o CDN). |
| `widget.js` | Código fuente. Después de cambiarlo, ejecuta `npm run build` para regenerar `widget.min.js`. |
| `demo.html` | Página de ejemplo / muestra comercial. |
| `INSTALACION.md` | Esta guía. |

---

## Puesta en marcha (una vez, de tu lado)

1. Sube `widget.min.js` a tu hosting o a un CDN. Ejemplos de URL final:
   - `https://tudominio.com/widget.min.js`
   - o un CDN gratis como jsDelivr apuntando a un repo de GitHub.
2. Esa URL es la que van a usar todos tus clientes. Cuando mejores el widget, lo actualizas
   ahí y **todos se actualizan solos**.

---

## Alta de un cliente nuevo

### Paso 1 — Pegar el widget en el sitio del cliente

Antes de `</body>` (o en el `<head>`), una línea:

```html
<script src="https://tudominio.com/widget.min.js"
        data-phone="5491122334455"
        data-country="AR"
        data-color="#25D366"
        data-cta-text="Escríbenos"></script>
```

Aparece la burbuja, el visitante deja sus datos, se publica `walead_submit` en el `dataLayer`
y se abre WhatsApp con el mensaje armado.

### Paso 2 — Configurar GTM

Ver [Medición y conversiones avanzadas](#medición-y-conversiones-avanzadas-google-ads--ga4--meta).
Ahí también se arma el guardado en la hoja o el CRM.

---

## Todos los atributos disponibles

Solo **`data-phone` es obligatorio**. El resto son opcionales.

| Atributo | Ejemplo | Descripción |
|---|---|---|
| `data-phone` | `5491122334455` | **(Obligatorio)** Número de WhatsApp del negocio, con código de país, sin `+` ni espacios. |
| `data-name` | `Pizzería Don José` | Nombre del negocio. Es el título por defecto y se inserta con `{negocio}` en cualquier texto. |
| `data-country` | `AR` | País preseleccionado en el teléfono (código ISO). Si lo omites, se detecta por el idioma del navegador. |
| `data-countries` | `AR,UY,PY` | Países que aparecen destacados arriba de la lista (el resto sigue disponible). |
| `data-color` | `#25D366` | Color de marca (burbuja, botón, encabezado). |
| `data-text-color` | `#ffffff` | Color del texto y los íconos sobre el color de marca (título, logo de WhatsApp, botón, burbuja). Con `auto` se elige negro o blanco según el contraste. |
| `data-theme` | `auto` / `light` / `dark` | Tema del panel. Por defecto `auto`: mira el fondo del sitio. La fuente del widget también se toma del sitio. |
| `data-position` | `right` / `left` | Esquina donde aparece la burbuja. Por defecto `right`. |
| `data-title` | `¿Hablamos?` | Título del encabezado del panel. |
| `data-subtitle` | `Te respondemos por WhatsApp` | Línea debajo del título. Por defecto: "Te respondemos por WhatsApp". Vacío (`data-subtitle=""`) = no se muestra. |
| `data-avatar` | `https://…/logo.png` | Logo del negocio en el encabezado, sobre un círculo blanco y con un puntito verde de WhatsApp. Conviene un logo cuadrado. Por defecto, el ícono de WhatsApp. |
| `data-greeting` | `¡Hola! 👋 …` | Mensaje tipo chat que se muestra arriba del formulario. Vacío = no se muestra. |
| `data-cta-text` | `¿Te ayudamos? 👋` | Globo de texto junto a la burbuja (si lo omites, muestra solo el ícono). |
| `data-button-text` | `Continuar en WhatsApp` | Texto del botón de envío. |
| `data-template` | ver abajo | Mensaje que queda escrito en WhatsApp. |
| `data-privacy` | `Te respondemos en el día.` | Texto opcional con candado debajo del botón. Por defecto no se muestra. |
| `data-delay` | `5000` | Milisegundos para abrir el panel solo (una vez por sesión). `0` = no abrir solo. |
| `data-thanks` | `Te estamos llevando…` | Texto de la pantalla de confirmación. |
| `data-triggers` | `.btn-contacto` | Selector CSS de botones/links del sitio que abren el widget. Ver abajo. |
| `data-launcher` | `false` | Oculta la burbuja flotante: el widget se abre solo desde botones del sitio. |

### Abrir el widget desde botones del sitio ("Contacto", "Consúltanos"…)

Hay tres formas; se pueden combinar.

**1. Marcar el botón** (si puedes editar el HTML del sitio):

```html
<button data-walead-open>Consúltanos</button>
<a href="#walead">Contacto</a>
```

**2. Usar los botones que ya existen** (sin tocar el sitio) con `data-triggers` y un selector CSS:

```html
<script src="…/widget.min.js"
        data-phone="5491122334455"
        data-triggers=".btn-contacto, #menu a[href='/contacto']"></script>
```

> Tip: `data-triggers="a[href*='wa.me'], a[href*='api.whatsapp.com']"` intercepta los links
> de WhatsApp que el sitio ya tenga, así esos clics también dejan el lead antes de ir al chat.

**3. Por JavaScript** (desde GTM, un popup, otro script):

```js
WaLead.open();   // abre
WaLead.close();  // cierra
WaLead.toggle(); // alterna
```

En los tres casos el clic no navega (si era un link, se cancela) y se abre el panel.
Cada apertura publica `dataLayer.push({event:"walead_open", walead_origen})`, con origen
`burbuja`, `boton`, `auto` o `api`. Ver [embudo](#embudo-de-conversión).

### El formulario

Pide solo dos datos, los que sirven para conversiones avanzadas:

- **Email**: se valida y se sugiere la corrección de errores comunes (`gmial.com` → `gmail.com`).
- **WhatsApp**: con selector de país (bandera + código, con buscador). El número se guarda
  siempre en formato internacional **E.164** (`+5491122334455`), aunque la persona lo escriba
  con 0, con 15, con espacios o pegue el número con `+código` (en ese caso el país se cambia solo).

Si la persona vuelve al sitio, el formulario aparece precargado con sus datos.

### Los dos mensajes: saludo y mensaje de WhatsApp

- **`data-greeting`**: lo que el negocio le dice al visitante dentro del widget.
- **`data-template`**: lo que el visitante te envía; queda escrito en el chat de WhatsApp
  (la persona lo puede editar antes de mandarlo).

Variables disponibles:

| Variable | Dónde | Se reemplaza por |
|---|---|---|
| `{negocio}` | todos los textos | el valor de `data-name` |
| `{pagina}` | mensaje de WhatsApp | título de la página donde estaba el visitante |
| `{url}` | mensaje de WhatsApp | link de esa página |
| `{email}`, `{phone}`, `{country}` | mensaje de WhatsApp | lo que cargó el visitante |
| `{tipo}` | mensaje de WhatsApp | tipo de cliente elegido: Consumidor Final, Institución o Revendedor |

```html
data-name="Pizzería Don José"
data-greeting="¡Hola! 👋 Somos {negocio}. Dejanos tu email y tu WhatsApp y te respondemos enseguida."
data-template="¡Hola {negocio}! Vi la página {pagina} y quiero hacer una consulta."
```

### Configurador visual

`configurador.html` arma la línea de instalación sin escribir código: completas nombre,
número, mensajes y colores, ves el widget y el mensaje de WhatsApp en vivo, y copias el código
(versión `<script>` o versión GTM). Ábrelo en el navegador desde el repo, o publícalo con
GitHub Pages para compartirlo con tus clientes.

### Instalación desde Google Tag Manager

En GTM (etiqueta HTML personalizada) los `data-` del script no siempre se pueden leer.
Define la configuración antes de cargar el widget:

```html
<script>
  window.WALEAD_CONFIG = {
    phone: "5491122334455",
    country: "AR"
  };
  (function () {
    var s = document.createElement("script");
    s.src = "https://victorbogado23.github.io/walead_calidadextrema/widget.min.js?d=" + new Date().toISOString().slice(0, 15);
    document.head.appendChild(s);
  })();
</script>
```

El `?d=` con la fecha y la hora hace que cada navegador pida el widget de nuevo cada
10 minutos, así nadie se queda con una versión vieja guardada.

Las claves son las mismas que los atributos, en camelCase (`buttonText`, `ctaText`, …).

### Actualizaciones automáticas

El widget se sirve desde GitHub Pages (rama `main`): carga siempre la última versión publicada,
así que las mejoras llegan solas a todos los sitios **sin tocar GTM**. Nunca se quitan ni se
renombran opciones ni datos del `dataLayer`; solo se agregan, para que nada de lo armado en GTM
se rompa.

GitHub Pages publica cada push a `main` en 1 o 2 minutos y lo guarda en caché 10 minutos; con el
código de GTM de arriba, cada navegador toma la versión nueva en 10 a 20 minutos como máximo.

Se dejó de usar jsDelivr (`@main`) porque tardaba horas en tomar los cambios y a veces volvía
a entregar versiones viejas aunque se purgara.

---

## Clientes recurrentes

Cada envío publica `walead_submit`, aunque la persona ya haya escrito antes: un cliente
recurrente que vuelve a consultar es una consulta nueva. El `event_id` de cada envío lo genera
GTM. `walead_lead_id` es estable por persona (sale del número del negocio, el email y el
teléfono) y sirve para reconocer al cliente recurrente.

Solo se frena el doble envío accidental: si en la misma página el mismo lead se envía otra vez
dentro de 1 minuto, se publica `walead_repeat` en lugar de `walead_submit`.

---

## Medición y conversiones avanzadas (Google Ads / GA4 / Meta)

Al enviarse un lead, el widget hace:

```js
dataLayer.push({
  event: "walead_submit",
  walead_lead_id: "wl_3f9a1c2e7b604d18",
  walead_tipo_cliente: "Revendedor", // Consumidor Final | Institución | Revendedor
  user_data: {
    email: "juan@gmail.com",           // en minúsculas
    phone_number: "+5491122334455",    // E.164
    address: { country: "AR" }
  }
});
```

El email y el teléfono van **solo** dentro de `user_data`.

El campo **Tipo de cliente** es obligatorio: el visitante elige entre Consumidor Final,
Institución o Revendedor, y el valor llega en `walead_tipo_cliente`.

Configuración en GTM:

1. **Activador**: Evento personalizado → nombre `walead_submit`.
2. **Variables de capa de datos** (versión 2): `walead_lead_id`, `walead_tipo_cliente`, `user_data.email`,
   `user_data.phone_number` y, si la quieres entera, `user_data`.
3. **Google Ads — Conversiones avanzadas**: en la etiqueta de conversión, activa *Incluir datos
   proporcionados por el usuario* → nueva variable *Datos proporcionados por el usuario* →
   *Configuración manual* → Email = `{{user_data.email}}`, Teléfono = `{{user_data.phone_number}}`.
   En *ID de transacción* usa `{{walead_lead_id}}`.
4. **GA4**: en la etiqueta de evento (ej. `generate_lead`), en *Datos proporcionados por el
   usuario*, usa la misma variable.
5. **Meta (píxel / CAPI)**: manda `event_id` = `{{walead_lead_id}}`, `em` = `{{user_data.email}}`
   y `ph` = `{{user_data.phone_number}}` (Meta pide el teléfono sin `+`; GTM o tu plantilla de
   CAPI lo normaliza y hashea).
6. **Hoja o CRM**: una etiqueta con el mismo activador que envíe `walead_lead_id`, el email,
   el teléfono y `walead_tipo_cliente`. Los datos de campaña (`gclid`, `utm_*`) los toma GTM con sus propias variables.

Google y Meta hashean (SHA-256) los datos del lado de la etiqueta: el widget los entrega
normalizados (email en minúsculas, teléfono E.164), que es lo que exigen para que coincidan.

### Embudo de conversión

Cada paso publica un evento en el `dataLayer`; crea en GTM un activador por cada uno:

| Evento | Cuándo ocurre | Se envía |
|---|---|---|
| `walead_open` | Se abre el panel | Una vez por apertura, con `walead_origen` |
| `walead_start` | La persona empieza a completar el formulario | Una vez por carga de página |
| `walead_submit` | Consulta enviada (también clientes recurrentes) | Es la conversión |
| `walead_repeat` | Doble envío del mismo lead en menos de 1 minuto | No es conversión |

---

## Notas

- El widget es liviano, sin dependencias, y no interfiere con el diseño del sitio: vive
  dentro de un Shadow DOM, así que ni los estilos del sitio lo rompen ni los suyos se filtran.
- Diseño adaptable: en celulares el panel sube desde abajo, con fondo oscurecido.
- Accesible: navegación con teclado (Tab, flechas en la lista de países, Esc para cerrar),
  etiquetas y errores leídos por lectores de pantalla, respeta "reducir movimiento".
- Las banderas se cargan desde `flagcdn.com`.
- Si `data-phone` falta, el widget no se muestra (y avisa por consola).
- El widget no envía los datos del visitante a ningún servidor: solo los publica en el
  `dataLayer` del propio sitio. Adónde van después lo decide la configuración de GTM.

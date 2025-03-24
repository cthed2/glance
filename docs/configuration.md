# Configuración de Glance

- [Página preconfigurada](#página-preconfigurada)
- [El archivo de configuración](#el-archivo-de-configuración)
  - [Recarga automática](#recarga-automática)
  - [Variables de entorno](#variables-de-entorno)
  - [Incluir otros archivos de configuración](#incluir-otros-archivos-de-configuración)
- [Servidor](#servidor)
- [Documento](#documento)
- [Marca](#marca)
- [Tema](#tema)
  - [Temas](#temas)
- [Páginas y Columnas](#páginas--columnas)
- [Widgets](#widgets)
  - [RSS](#rss)
  - [Vídeos](#vídeos)
  - [Hacker News](#hacker-news)
  - [Lobsters](#lobsters)
  - [Reddit](#reddit)
  - [Widget de Búsqueda](#widget-de-búsqueda)
  - [Grupo](#grupo)
  - [Columna Dividida](#columna-dividida)
  - [API Personalizada](#api-personalizada)
  - [Extensión](#extensión)
  - [Clima](#clima)
  - [Monitor](#monitor)
  - [Lanzamientos](#lanzamientos)
  - [Contenedores Docker](#contenedores-docker)
  - [Estadísticas DNS](#estadísticas-dns)
  - [Estadísticas del Servidor](#estadísticas-del-servidor)
  - [Repositorio](#repositorio)
  - [Marcadores](#marcadores)
  - [Calendario](#calendario)
  - [Calendario (legado)](#calendario-legado)
  - [ChangeDetection.io](#changedetectionio)
  - [Reloj](#reloj)
  - [Mercados](#mercados)
  - [Canales de Twitch](#canales-de-twitch)
  - [Juegos Top de Twitch](#juegos-top-de-twitch)
  - [iframe](#iframe)
  - [HTML](#html)


## Página preconfigurada
Si no quieres dedicar tiempo a leer todas las opciones de configuración disponibles y solo quieres algo para empezar rápidamente, puedes usar [este archivo `glance.yml`](glance.yml) y hacerle cambios según te convenga. Te dará una página que se parece a lo siguiente:

![](images/preconfigured-page-preview.png)

Configura los widgets, añade más, añade páginas extra, etc. ¡Hazlo tuyo!

## El archivo de configuración

### Recarga automática
Se soporta la recarga automática de la configuración, lo que significa que puedes hacer cambios en el archivo de configuración y que surtan efecto al guardar sin tener que reiniciar el contenedor/servicio. Los cambios en las variables de entorno no activan una recarga y requieren un reinicio manual. Borrar un archivo de configuración hará que ese archivo deje de ser vigilado, incluso si se vuelve a crear.

> [!NOTA]
>
> Si intentas iniciar Glance con una configuración inválida, saldrá con un error directamente. Si has iniciado Glance con éxito con una configuración válida y luego le has hecho cambios que resultan en un error, verás ese error en la consola y Glance seguirá funcionando con la configuración antigua. Entonces puedes seguir haciendo cambios y cuando no haya errores, la nueva configuración se cargará.

> [!PRECAUCIÓN]
>
> Recargar el archivo de configuración borra tus datos en caché, lo que significa que tienes que solicitar los datos de nuevo cada vez que haces esto. Esto puede llevar a la limitación de velocidad para algunas APIs si lo haces con demasiada frecuencia. En el futuro se añadirá una caché que persista entre recargas.

### Variables de entorno
Se soporta la inserción de variables de entorno en cualquier parte de la configuración. Esto se hace a través de la sintaxis `${ENV_VAR}`. Intentar usar una variable de entorno que no existe resultará en un error y Glance o bien no se iniciará o no cargará tu nueva configuración al guardar. Ejemplo:

```yaml
server:
  host: ${HOST}
  port: ${PORT}
```

También puede estar en medio de una cadena:

```yaml
- type: rss
  title: ${RSS_TITLE}
  feeds:
    - url: http://domain.com/rss/${RSS_CATEGORY}.xml
```

Funciona con cualquier tipo de valor, no solo con cadenas:

```yaml
- type: rss
  limit: ${RSS_LIMIT}
```

Si necesitas usar la sintaxis `${NAME}` en tu configuración sin que se interprete como una variable de entorno, puedes escaparla prefijándola con una barra invertida `\`:

```yaml
something: \${NOT_AN_ENV_VAR}
```

### Incluir otros archivos de configuración
Se soporta la inclusión de archivos de configuración desde dentro de tu archivo de configuración principal. Esto se hace a través de la directiva `!include` junto con una ruta relativa o absoluta al archivo que quieres incluir. Si la ruta es relativa, será relativa al archivo de configuración principal. Además, se pueden usar variables de entorno dentro de los archivos incluidos, y los cambios en los archivos incluidos activarán una recarga automática. Ejemplo:

```yaml
pages:
  !include: home.yml
  !include: videos.yml
  !include: homelab.yml
```

El archivo que estás incluyendo no debe tener ninguna indentación adicional, sus valores deben estar en el nivel superior y la cantidad apropiada de indentación se añadirá automáticamente dependiendo de dónde se incluya el archivo. Ejemplo:

`glance.yml`

```yaml
pages:
  - name: Home
    columns:
      - size: full
        widgets:
          !include: rss.yml
  - name: News
    columns:
      - size: full
        widgets:
          - type: group
            widgets:
              !include: rss.yml
              - type: reddit
                subreddit: news
```

`rss.yml`

```yaml
- type: rss
  title: News
  feeds:
    - url: ${RSS_URL}
```

La directiva `!include` se puede usar en cualquier parte del archivo de configuración, no solo en la propiedad `pages`, sin embargo, debe estar en su propia línea y tener la indentación apropiada.

Si te encuentras con errores de análisis YAML al usar la directiva `!include`, es probable que los números de línea informados sean incorrectos. Esto se debe a que la inclusión de archivos se hace antes de que se analice el YAML, ya que YAML en sí no soporta la inclusión de archivos. Para ayudar a la depuración en casos como este, puedes usar el comando `config:print` y pasarlo a `less -N` para ver el archivo de configuración completo con las inclusiones resueltas y los números de línea añadidos:

```sh
glance --config /ruta/a/glance.yml config:print | less -N
```

Esto es un poco más complicado cuando se ejecuta Glance dentro de un contenedor Docker:

```sh
docker run --rm -v ./glance.yml:/app/config/glance.yml glanceapp/glance config:print | less -N
```

Esto asume que la configuración que quieres imprimir está en tu directorio de trabajo actual y se llama `glance.yml`.

## Servidor
La configuración del servidor se hace a través de una propiedad `server` de nivel superior. Ejemplo:

```yaml
server:
  port: 8080
  assets-path: /home/user/glance-assets
```

### Propiedades

| Nombre | Tipo | Requerido | Predeterminado |
| ---- | ---- | -------- | ------- |
| host | string | no |  |
| port | number | no | 8080 |
| base-url | string | no | |
| assets-path | string | no |  |

#### `host`
La dirección en la que el servidor escuchará. Establecerlo en `localhost` significa que solo la máquina en la que se está ejecutando el servidor podrá acceder al dashboard. Por defecto, escuchará en todas las interfaces.

#### `port`
Un número entre 1 y 65.535, siempre y cuando ese puerto no esté ya siendo usado por otra cosa.

#### `base-url`
La URL base bajo la que se aloja Glance. No es necesario especificar esto a menos que estés usando un proxy inverso y estés alojando Glance bajo un directorio. Si ese es el caso, entonces puedes establecer este valor a `/glance` o como se llame el directorio. Ten en cuenta que la barra inclinada (`/`) al principio es requerida a menos que especifiques el dominio completo y la ruta.

> [!IMPORTANTE]
> Necesitas quitar el prefijo `base-url` antes de reenviar la petición al servidor de Glance.
> En Caddy puedes hacer esto usando [`handle_path`](https://caddyserver.com/docs/caddyfile/directives/handle_path) o [`uri strip_prefix`](https://caddyserver.com/docs/caddyfile/directives/uri).

#### `assets-path`
La ruta a un directorio que será servido por el servidor bajo la ruta `/assets/`. Esto es útil para widgets como el Monitor donde tienes que especificar una URL de icono y quieres auto-alojar todos los iconos en lugar de apuntar a una fuente externa.

> [!IMPORTANTE]
>
> Al instalar a través de docker la ruta apuntará a los archivos dentro del contenedor. No olvides montar tu ruta de assets a la misma ruta dentro del contenedor.
> Ejemplo:
>
> Si tus assets están en:
> ```
> /home/user/glance-assets
> ```
>
> Deberías montar:
> ```
> /home/user/glance-assets:/app/assets
> ```
>
> Y tu configuración debería contener:
> ```
> assets-path: /app/assets
> ```

##### Ejemplos

Digamos que tienes un directorio `glance-assets` con un archivo `gitea-icon.png` dentro y especificas tu ruta de assets como:

```yaml
assets-path: /home/user/glance-assets
```

Para poder apuntar a un asset desde tu ruta de assets, usa la ruta `/assets/` así:

```yaml
icon: /assets/gitea-icon.png
```

## Documento
Si quieres insertar HTML personalizado en el `<head>` del documento para todas las páginas, puedes hacerlo usando la propiedad `document`. Ejemplo:

```yaml
document:
  head: |
    <script src="/assets/custom.js"></script>
```

## Marca
Puedes ajustar varias partes de la marca a través de una propiedad `branding` de nivel superior. Ejemplo:

```yaml
branding:
  custom-footer: |
    <p>Funciona con <a href="https://github.com/glanceapp/glance">Glance</a></p>
  logo-url: /assets/logo.png
  favicon-url: /assets/logo.png
```

### Propiedades

| Nombre | Tipo | Requerido | Predeterminado |
| ---- | ---- | -------- | ------- |
| hide-footer | bool | no | false |
| custom-footer | string | no |  |
| logo-text | string | no | G |
| logo-url | string | no | |
| favicon-url | string | no | |

#### `hide-footer`
Oculta el pie de página cuando se establece en `true`.

#### `custom-footer`
Especifica HTML personalizado para usar en el pie de página.

#### `logo-text`
Especifica texto personalizado para usar en lugar de la "G" que se encuentra en la navegación.

#### `logo-url`
Especifica una URL a una imagen personalizada para usar en lugar de la "G" que se encuentra en la navegación. Si se establecen tanto `logo-text` como `logo-url`, solo se usará `logo-url`.

#### `favicon-url`
Especifica una URL a una imagen personalizada para usar como favicon.

## Tema
La tematización se hace a través de una propiedad `theme` de nivel superior. Los valores para los colores están en formato [HSL](https://giggster.com/guide/basics/hue-saturation-lightness/) (tono, saturación, luminosidad). Puedes usar un selector de color [como este](https://hslpicker.com/) para convertir colores de otros formatos a HSL. Los valores están separados por un espacio y no se requiere `%` para ninguno de los números.

Ejemplo:

```yaml
theme:
  background-color: 100 20 10
  primary-color: 40 90 40
  contrast-multiplier: 1.1
```

### Temas
Si no quieres dedicar tiempo a configurar tu propio tema, hay [varios temas disponibles](themes.md) de los que puedes simplemente copiar los valores.

### Propiedades
| Nombre | Tipo | Requerido | Predeterminado |
| ---- | ---- | -------- | ------- |
| light | boolean | no | false |
| background-color | HSL | no | 240 8 9 |
| primary-color | HSL | no | 43 50 70 |
| positive-color | HSL | no | igual que `primary-color` |
| negative-color | HSL | no | 0 70 70 |
| contrast-multiplier | number | no | 1 |
| text-saturation-multiplier | number | no | 1 |
| custom-css-file | string | no | |

#### `light`
Si el esquema es claro u oscuro. Esto no cambia el color de fondo, invierte los colores del texto para que se vean apropiadamente en un fondo claro.

#### `background-color`
Color de la página y los widgets.

#### `primary-color`
Color usado en toda la página, principalmente para indicar enlaces no visitados.

#### `positive-color`
Usado para indicar que algo es positivo, como que el precio de las acciones sube, que un canal de Twitch está en directo o que un sitio monitorizado está online. Si no se establece, se usará el valor de `primary-color`.

#### `negative-color`
Opuesto a `positive-color`.

#### `contrast-multiplier`
Usado para aumentar o disminuir el contraste (en otras palabras, la visibilidad) del texto. Un valor de `1.3` significa que el texto será un 30% más claro/oscuro dependiendo del esquema. Usa esto si crees que parte del texto en la página es demasiado oscuro y difícil de leer. Ejemplo:

![diferencia entre contraste 1 y 1.3](images/contrast-multiplier-example.png)

#### `text-saturation-multiplier`
Usado para aumentar o disminuir la saturación del texto, útil cuando se usa un color de fondo personalizado con una gran cantidad de saturación y se necesita que el texto tenga un color más neutro. `0.5` significa que la saturación será un 50% menor y `1.5` significa que será un 50% mayor.

#### `custom-css-file`
Ruta a un archivo CSS personalizado, ya sea externo o uno dentro de la ruta de assets configurada del servidor. Ejemplo:

```yaml
theme:
  custom-css-file: /assets/my-style.css
```

> [!CONSEJO]
>
> Debido a que Glance usa muchas clases de utilidad, puede ser difícil apuntar a algunos elementos. Para facilitar el estilo de widgets específicos, cada widget tiene una clase `widget-type-{name}`, así que por ejemplo, si quisieras hacer los enlaces dentro solo del widget RSS más grandes, podrías usar el siguiente selector:
>
> ```css
> .widget-type-rss a {
>     font-size: 1.5rem;
> }
> ```
>
> Además, también puedes usar la propiedad `css-class` que está disponible en cada widget para establecer nombres de clase personalizados para widgets individuales.


## Páginas y Columnas
![ilustración de páginas y columnas](images/pages-and-columns-illustration.png)

Usar páginas y columnas es cómo se organizan los widgets. Cada página contiene hasta 3 columnas y cada columna puede tener cualquier número de widgets.

### Páginas
Las páginas se definen a través de una propiedad `pages` de nivel superior. La página definida primero se convierte en la página de inicio y todas las páginas se añaden automáticamente a la barra de navegación en el orden en que se definieron. Ejemplo:

```yaml
pages:
  - name: Home
    columns: ...

  - name: Videos
    columns: ...

  - name: Homelab
    columns: ...
```

### Propiedades
| Nombre | Tipo | Requerido | Predeterminado |
| ---- | ---- | -------- | ------- |
| name | string | sí | |
| slug | string | no | |
| width | string | no | |
| center-vertically | boolean | no | false |
| hide-desktop-navigation | boolean | no | false |
| expand-mobile-page-navigation | boolean | no | false |
| show-mobile-header | boolean | no | false |
| columns | array | sí | |

#### `title`
El nombre de la página que se muestra en la barra de navegación.

#### `slug`
La versión amigable para URL del título que se usa para acceder a la página. Por ejemplo, si el título de la página es "RSS Feeds" puedes hacer que la página sea accesible a través de `localhost:8080/feeds` estableciendo el slug a `feeds`. Si no se define, se generará automáticamente a partir del título.

#### `width`
El ancho máximo de la página en el escritorio. Los valores posibles son `slim` y `wide`.

* predeterminado: `1600px` (cuando no se especifica ningún valor)
* slim: `1100px`
* wide: `1920px`

> [!NOTA]
>
> Al usar `slim`, el número máximo de columnas permitidas para esa página es `2`.

#### `center-vertically`
Cuando se establece en `true`, centra verticalmente el contenido en la página. No tiene efecto si el contenido es más alto que la altura del viewport.

#### `hide-desktop-navigation`
Si se deben mostrar los enlaces de navegación en la parte superior de la página en el escritorio.

#### `expand-mobile-page-navigation`
Si la navegación de la página móvil debe estar expandida por defecto.

#### `show-mobile-header`
Si se debe mostrar un encabezado mostrando el nombre de la página en el móvil. El encabezado tiene a propósito mucho espacio en blanco vertical para empujar el contenido hacia abajo y hacerlo más fácil de alcanzar en dispositivos altos.

Vista previa:

![](images/mobile-header-preview.png)

### Columnas
Las columnas se definen para cada página usando una propiedad `columns`. Hay dos tipos de columnas: `full` y `small`, que se refiere a su ancho. Una columna pequeña ocupa una cantidad fija de ancho (300px) y una columna completa ocupa todo el ancho restante. Puedes tener hasta 3 columnas por página y debes tener o bien 1 o 2 columnas completas. Ejemplo:

```yaml
pages:
  - name: Home
    columns:
      - size: small
        widgets: ...
      - size: full
        widgets: ...
      - size: small
        widgets: ...
```

### Propiedades
| Nombre | Tipo | Requerido |
| ---- | ---- | -------- |
| size | string | sí |
| widgets | array | no |

Aquí tienes algunas de las configuraciones de columna posibles:

![configuración de columna pequeña-completa-pequeña](images/column-configuration-1.png)

```yaml
columns:
  - size: small
    widgets: ...
  - size: full
    widgets: ...
  - size: small
    widgets: ...
```

![configuración de columna pequeña-completa-pequeña](images/column-configuration-2.png)

```yaml
columns:
  - size: full
    widgets: ...
  - size: small
    widgets: ...
```

![configuración de columna pequeña-completa-pequeña](images/column-configuration-3.png)

```yaml
columns:
  - size: full
    widgets: ...
  - size: full
    widgets: ...
```

## Widgets
Los widgets se definen para cada columna usando una propiedad `widgets`. Ejemplo:

```yaml
pages:
  - name: Home
    columns:
      - size: small
        widgets:
          - type: weather
            location: London, United Kingdom
```

> [!NOTA]
>
> Actualmente no todos los widgets están diseñados para encajar en todos los tamaños de columna, sin embargo, algunos widgets ofrecen diferentes "estilos" que ayudan a paliar esta limitación.

### Propiedades Compartidas
| Nombre | Tipo | Requerido |
| ---- | ---- | -------- |
| type | string | sí |
| title | string | no |
| title-url | string | no |
| cache | string | no |
| css-class | string | no |

#### `type`
Usado para especificar el widget.

#### `title`
El título del widget. Si se deja en blanco, lo definirá el widget.

#### `title-url`
La URL a la que ir cuando se hace clic en el título del widget. Si se deja en blanco, lo definirá el widget (si está disponible).

#### `cache`
Cuánto tiempo mantener los datos recuperados en memoria. El valor es una cadena y debe ser un número seguido de una de las letras s, m, h, d. Ejemplos:

```yaml
cache: 30s # 30 segundos
cache: 5m  # 5 minutos
cache: 2h  # 2 horas
cache: 1d  # 1 día
```

> [!NOTA]
>
> No todos los widgets pueden tener su duración de caché modificada. Los widgets de calendario y clima se actualizan cada hora y esto no se puede cambiar.

#### `css-class`
Establece clases CSS personalizadas para la instancia de widget específica.

### RSS
Muestra una lista de artículos de múltiples feeds RSS.

Ejemplo:

```yaml
- type: rss
  title: Noticias
  style: horizontal-cards
  feeds:
    - url: https://feeds.bloomberg.com/markets/news.rss
      title: Bloomberg
    - url: https://moxie.foxbusiness.com/google-publisher/markets.xml
      title: Fox Business
    - url: https://moxie.foxbusiness.com/google-publisher/technology.xml
      title: Fox Business
```

#### Propiedades
| Nombre | Tipo | Requerido | Predeterminado |
| ---- | ---- | -------- | ------- |
| style | string | no | vertical-list |
| feeds | array | sí |
| thumbnail-height | float | no | 10 |
| card-height | float | no | 27 |
| limit | integer | no | 25 |
| preserve-order | bool | no | false |
| single-line-titles | boolean | no | false |
| collapse-after | integer | no | 5 |

##### `limit`
El número máximo de artículos a mostrar.

##### `collapse-after`
Cuántos artículos son visibles antes de que aparezca el botón "MOSTRAR MÁS". Establecer a `-1` para que nunca se colapse.

##### `preserve-order`
Cuando se establece en `true`, el orden de los artículos se preservará tal como están en los feeds. Útil si un feed usa su propio orden de clasificación que denota la importancia de los artículos. Si usas esta propiedad teniendo muchos feeds, se recomienda establecer un `limit` para cada feed individual ya que si el primer feed definido tiene 15 artículos, los artículos del segundo feed comenzarán después del artículo 15 en la lista.

##### `single-line-titles`
Cuando se establece en `true`, trunca el título de cada publicación si excede una línea. Solo se aplica cuando el estilo se establece en `vertical-list`.

##### `style`
Usado para cambiar la apariencia del widget. Los valores posibles son:

* `vertical-list` - adecuado para columnas `full` y `small`
* `detailed-list` - adecuado para columnas `full`
* `horizontal-cards` - adecuado para columnas `full`
* `horizontal-cards-2` - adecuado para columnas `full`

A continuación se muestra una vista previa de cada estilo:

`vertical-list`

![vista previa del estilo vertical-list para el widget RSS](images/rss-feed-vertical-list-preview.png)

`detailed-list`

![vista previa del estilo detailed-list para el widget RSS](images/rss-widget-detailed-list-preview.png)

`horizontal-cards`

![vista previa del estilo horizontal-cards para el widget RSS](images/rss-feed-horizontal-cards-preview.png)

`horizontal-cards-2`

![vista previa del estilo horizontal-cards-2 para el widget RSS](images/rss-widget-horizontal-cards-2-preview.png)

##### `thumbnail-height`
Usado para modificar la altura de las miniaturas. Solo funciona cuando el estilo se establece en `horizontal-cards`. El valor predeterminado es `10` y las unidades son `rem`, si quieres por ejemplo duplicar la altura de las miniaturas puedes establecerlo en `20`.

##### `card-height`
Usado para modificar la altura de las tarjetas cuando se usa el estilo `horizontal-cards-2`. El valor predeterminado es `27` y las unidades son `rem`.

##### `feeds`
Un array de feeds RSS/atom. El título se puede cambiar opcionalmente.

###### Propiedades para cada feed
| Nombre | Tipo | Requerido | Predeterminado | Notas |
| ---- | ---- | -------- | ------- | ----- |
| url | string | sí | | |
| title | string | no | el título proporcionado por el feed | |
| hide-categories | boolean | no | false | Solo aplicable para el estilo `detailed-list` |
| hide-description | boolean | no | false | Solo aplicable para el estilo `detailed-list` |
| limit | integer | no | | |
| item-link-prefix | string | no | | |
| headers | key (string) & value (string) | no | | |

###### `limit`
El número máximo de artículos a mostrar desde ese feed específico. Útil si tienes un feed que publica muchos artículos con frecuencia y quieres evitar que empuje excesivamente hacia abajo los artículos de otros feeds.

###### `item-link-prefix`
Si un feed RSS no está devolviendo enlaces de artículo con un dominio base y Glance no ha podido detectar automáticamente el dominio correcto, puedes añadir manualmente un prefijo a cada enlace con esta propiedad.

###### `headers`
Opcionalmente, especifica los encabezados que se enviarán con la petición. Ejemplo:

```yaml
- type: rss
  feeds:
    - url: https://domain.com/rss
      headers:
        User-Agent: Agente de Usuario Personalizado
```

### Vídeos
Muestra una lista de los últimos vídeos de canales específicos de YouTube.

Ejemplo:

```yaml
- type: videos
  channels:
    - UCXuqSBlHAE6Xw-yeJA0Tunw
    - UCBJycsmduvYEL83R_U4JriQ
    - UCHnyfMqiRRG1u-2MsSQLbXA
```

Vista previa:
![](images/videos-widget-preview.png)

#### Propiedades
| Nombre | Tipo | Requerido | Predeterminado |
| ---- | ---- | -------- | ------- |
| channels | array | sí | |
| playlists | array | no | |
| limit | integer | no | 25 |
| style | string | no | horizontal-cards |
| collapse-after | integer | no | 7 |
| collapse-after-rows | integer | no | 4 |
| include-shorts | boolean | no | false |
| video-url-template | string | no | https://www.youtube.com/watch?v={VIDEO-ID} |

##### `channels`
Una lista de IDs de canales.

Una forma de obtener el ID de un canal es ir a la página del canal y hacer clic en su descripción:

![](images/videos-channel-description-example.png)

Luego desplázate hacia abajo y haz clic en "Compartir canal", luego en "Copiar ID de canal":

![](images/videos-copy-channel-id-example.png)

##### `playlists`

Una lista de IDs de listas de reproducción:

```yaml
- type: videos
  playlists:
    - PL8mG-RkN2uTyZZ00ObwZxxoG_nJbs3qec
    - PL8mG-RkN2uTxTK4m_Vl2dYR9yE41kRdBg
```

##### `limit`
El número máximo de vídeos a mostrar.

##### `collapse-after`
Especifica el número de vídeos a mostrar cuando se usa el estilo `vertical-list` antes de que aparezca el botón "MOSTRAR MÁS".

##### `collapse-after-rows`
Especifica el número de filas a mostrar cuando se usa el estilo `grid-cards` antes de que aparezca el botón "MOSTRAR MÁS".

##### `style`
Usado para cambiar la apariencia del widget. Los valores posibles son `horizontal-cards`, `vertical-list` y `grid-cards`.

Vista previa de `vertical-list`:

![](images/videos-widget-vertical-list-preview.png)

Vista previa de `grid-cards`:

![](images/videos-widget-grid-cards-preview.png)

##### `video-url-template`
Usado para reemplazar el enlace predeterminado para los vídeos. Útil cuando estás ejecutando tu propio front-end de YouTube. Ejemplo:

```yaml
video-url-template: https://invidious.your-domain.com/watch?v={VIDEO-ID}
```

Placeholders:

`{VIDEO-ID}` - el ID del vídeo

### Hacker News
Muestra una lista de publicaciones de [Hacker News](https://news.ycombinator.com/).

Ejemplo:

```yaml
- type: hacker-news
  limit: 15
  collapse-after: 5
```

Vista previa:
![](images/hacker-news-widget-preview.png)

#### Propiedades
| Nombre | Tipo | Requerido | Predeterminado |
| ---- | ---- | -------- | ------- |
| limit | integer | no | 15 |
| collapse-after | integer | no | 5 |
| comments-url-template | string | no | https://news.ycombinator.com/item?id={POST-ID} |
| sort-by | string | no | top |
| extra-sort-by | string | no | |

##### `comments-url-template`
Usado para reemplazar el enlace predeterminado para los comentarios de las publicaciones. Útil si quieres usar un front-end alternativo. Ejemplo:

```yaml
comments-url-template: https://www.hckrnws.com/stories/{POST-ID}
```

Placeholders:

`{POST-ID}` - el ID de la publicación

##### `sort-by`
Usado para especificar el orden en que se deben devolver las publicaciones. Los valores posibles son `top`, `new` y `best`.

##### `extra-sort-by`
Se puede usar para especificar una clasificación adicional que se aplicará sobre las publicaciones ya clasificadas. Por defecto no aplica ninguna clasificación extra y la única opción disponible es `engagement`.

La clasificación `engagement` intenta colocar las publicaciones con más puntos y comentarios en la parte superior, también priorizando las publicaciones recientes sobre las antiguas.

### Lobsters
Muestra una lista de publicaciones de [Lobsters](https://lobste.rs).

Ejemplo:

```yaml
- type: lobsters
  sort-by: hot
  tags:
    - go
    - security
    - linux
  limit: 15
  collapse-after: 5
```

Vista previa:
![](images/lobsters-widget-preview.png)

#### Propiedades
| Nombre | Tipo | Requerido | Predeterminado |
| ---- | ---- | -------- | ------- |
| instance-url | string | no | https://lobste.rs/ |
| custom-url | string | no | |
| limit | integer | no | 15 |
| collapse-after | integer | no | 5 |
| sort-by | string | no | hot |
| tags | array | no | |

##### `instance-url`
La URL base para una instancia de Lobsters alojada en otro lugar que no sea lobste.rs. Ejemplo:

```yaml
instance-url: https://www.journalduhacker.net/
```

##### `custom-url`
Una URL personalizada para recuperar publicaciones de Lobsters. Si se especifica esto, las propiedades `instance-url`, `sort-by` y `tags` se ignoran.

##### `limit`
El número máximo de publicaciones a mostrar.

##### `collapse-after`
Cuántas publicaciones son visibles antes de que aparezca el botón "MOSTRAR MÁS". Establecer a `-1` para que nunca se colapse.

##### `sort-by`
El orden de clasificación en el que se devuelven las publicaciones. Las opciones posibles son `hot` y `new`.

##### `tags`
Limita a las publicaciones que contengan una de las etiquetas dadas. **No puedes especificar un orden de clasificación cuando filtras por etiquetas, se establecerá por defecto a `hot`.**

### Reddit
Muestra una lista de publicaciones de un subreddit específico.

> [!ADVERTENCIA]
>
> Reddit no permite el acceso API no autorizado desde IPs de VPS, si estás alojando Glance en un VPS obtendrás una respuesta 403. Como solución alternativa, puedes enrutar el tráfico desde Glance a través de una VPN o tu propio proxy HTTP usando la propiedad `request-url-template`.

Ejemplo:

```yaml
- type: reddit
  subreddit: technology
```

#### Propiedades
| Nombre | Tipo | Requerido | Predeterminado |
| ---- | ---- | -------- | ------- |
| subreddit | string | sí |  |
| style | string | no | vertical-list |
| show-thumbnails | boolean | no | false |
| show-flairs | boolean | no | false |
| limit | integer | no | 15 |
| collapse-after | integer | no | 5 |
| comments-url-template | string | no | https://www.reddit.com/{POST-PATH} |
| request-url-template | string | no |  |
| proxy | string or multiple parameters | no |  |
| sort-by | string | no | hot |
| top-period | string | no | day |
| search | string | no | |
| extra-sort-by | string | no | |

##### `subreddit`
El subreddit del que se deben obtener las publicaciones.

##### `style`
Usado para cambiar la apariencia del widget. Los valores posibles son `vertical-list`, `horizontal-cards` y `vertical-cards`. Los dos primeros fueron diseñados para columnas completas y el último para columnas pequeñas.

`vertical-list`

![](images/reddit-widget-preview.png)

`horizontal-cards`

![](images/reddit-widget-horizontal-cards-preview.png)

`vertical-cards`

![](images/reddit-widget-vertical-cards-preview.png)

##### `show-thumbnails`
Muestra u oculta las miniaturas junto a la publicación. Esto solo funciona si el `style` es `vertical-list`. Vista previa:

![](images/reddit-widget-vertical-list-thumbnails.png)

> [!NOTA]
>
> Las miniaturas no funcionan para algunos subreddits debido a que la API de Reddit no devuelve la URL de la miniatura. Todavía no hay solución para esto.

##### `show-flairs`
Muestra los flairs de las publicaciones cuando se establece en `true`.

##### `limit`
El número máximo de publicaciones a mostrar.

##### `collapse-after`
Cuántas publicaciones son visibles antes de que aparezca el botón "MOSTRAR MÁS". Establecer a `-1` para que nunca se colapse. No disponible cuando se usan los estilos `vertical-cards` y `horizontal-cards`.

##### `comments-url-template`
Usado para reemplazar el enlace predeterminado para los comentarios de las publicaciones. Útil si quieres usar el diseño antiguo de Reddit o cualquier otro front-end de terceros. Ejemplo:

```yaml
comments-url-template: https://old.reddit.com/{POST-PATH}
```

Placeholders:

`{POST-PATH}` - la ruta completa a la publicación, como:

```
r/selfhosted/comments/bsp01i/welcome_to_rselfhosted_please_read_this_first/
```

`{POST-ID}` - el ID que viene después de `/comments/`

`{SUBREDDIT}` - el nombre del subreddit

##### `request-url-template`
Una URL de petición personalizada que se usará para obtener los datos. Esto es útil cuando estás alojando Glance en un VPS donde Reddit está bloqueando las peticiones y quieres enrutarlas a través de un proxy que acepte la URL como parte de la ruta o como parámetro de consulta.

Placeholders:

`{REQUEST-URL}` - se planteará y se reemplazará con la URL de petición expandida (es decir, https://www.reddit.com/r/selfhosted/hot.json). Ejemplo:

```
https://proxy/{REQUEST-URL}
https://your.proxy/?url={REQUEST-URL}
```

##### `proxy`
Una URL de proxy HTTP/HTTPS personalizada que se usará para obtener los datos. Esto es útil cuando estás alojando Glance en un VPS donde Reddit está bloqueando las peticiones y quieres evitar la restricción enrutando las peticiones a través de un proxy. Ejemplo:

```yaml
proxy: http://usuario:contraseña@proxy.com:8080
proxy: https://usuario:contraseña@proxy.com:443
```

Alternativamente, puedes especificar la URL del proxy así como opciones adicionales usando múltiples parámetros:

```yaml
proxy:
  url: http://proxy.com:8080
  allow-insecure: true
  timeout: 10s
```

###### `allow-insecure`
Cuando se establece en `true`, permite el uso de conexiones inseguras como cuando el proxy tiene un certificado autofirmado.

###### `timeout`
El tiempo máximo de espera para una respuesta del proxy. El valor es una cadena y debe ser un número seguido de una de las letras s, m, h, d. Ejemplo: `10s` para 10 segundos, `1m` para 1 minuto, etc.

##### `sort-by`
Se puede usar para especificar el orden en que se deben devolver las publicaciones. Los valores posibles son `hot`, `new`, `top` y `rising`.

##### `top-period`
Disponible solo cuando `sort-by` se establece en `top`. Los valores posibles son `hour`, `day`, `week`, `month`, `year` y `all`.

##### `search`
Palabras clave para buscar. También es posible buscar dentro de campos específicos, **aunque ten en cuenta que Reddit puede eliminar la capacidad de usar cualquiera de estos en cualquier momento**:

![](images/reddit-field-search.png)

##### `extra-sort-by`
Se puede usar para especificar una clasificación adicional que se aplicará sobre las publicaciones ya clasificadas. Por defecto no aplica ninguna clasificación extra y la única opción disponible es `engagement`.

La clasificación `engagement` intenta colocar las publicaciones con más puntos y comentarios en la parte superior, también priorizando las publicaciones recientes sobre las antiguas.

### Widget de Búsqueda
Muestra una barra de búsqueda que se puede usar para buscar términos específicos en varios motores de búsqueda.

Ejemplo:

```yaml
- type: search
  search-engine: duckduckgo
  bangs:
    - title: YouTube
      shortcut: "!yt"
      url: https://www.youtube.com/results?search_query={QUERY}
```

Vista previa:

![](images/search-widget-preview.png)

#### Atajos de teclado
| Teclas | Acción | Condición |
| ---- | ------ | --------- |
| <kbd>S</kbd> | Enfoca la barra de búsqueda | No está ya enfocado en otro campo de entrada |
| <kbd>Enter</kbd> | Realiza la búsqueda en la misma pestaña | La entrada de búsqueda está enfocada y no está vacía |
| <kbd>Ctrl</kbd> + <kbd>Enter</kbd> | Realiza la búsqueda en una nueva pestaña | La entrada de búsqueda está enfocada y no está vacía |
| <kbd>Escape</kbd> | Deja de enfocar | La entrada de búsqueda está enfocada |
| <kbd>Flecha arriba</kbd> | Inserta la última consulta de búsqueda desde que se abrió la página en el campo de entrada | La entrada de búsqueda está enfocada |

> [!CONSEJO]
>
> Puedes usar la propiedad `new-tab` con un valor de `true` si quieres mostrar los resultados de búsqueda en una nueva pestaña por defecto. <kbd>Ctrl</kbd> + <kbd>Enter</kbd> entonces mostrará los resultados en la misma pestaña.

#### Propiedades
| Nombre | Tipo | Requerido | Predeterminado |
| ---- | ---- | -------- | ------- |
| search-engine | string | no | duckduckgo |
| new-tab | boolean | no | false |
| autofocus | boolean | no | false |
| placeholder | string | no | Escribe aquí para buscar… |
| bangs | array | no | |

##### `search-engine`
O bien un valor de la tabla de abajo o una URL a un motor de búsqueda personalizado. Usa `{QUERY}` para indicar dónde se coloca el valor de la consulta.

| Nombre | URL |
| ---- | --- |
| duckduckgo | `https://duckduckgo.com/?q={QUERY}` |
| google | `https://www.google.com/search?q={QUERY}` |

##### `new-tab`
Cuando se establece en `true`, intercambia los atajos para mostrar los resultados en la misma pestaña o en una nueva, estableciendo por defecto mostrar los resultados en una nueva pestaña.

##### `autofocus`
Cuando se establece en `true`, enfoca automáticamente la entrada de búsqueda al cargar la página.

##### `placeholder`
Cuando se establece, modifica el texto mostrado en el campo de entrada antes de escribir.

##### `bangs`
¿Qué ahora? [Bangs](https://duckduckgo.com/bangs). Son atajos que te permiten usar la misma caja de búsqueda para muchos sitios diferentes. Asumiendo que lo tienes configurado, si por ejemplo empiezas tu entrada de búsqueda con `!yt` podrías realizar una búsqueda en YouTube:

![](images/search-widget-bangs-preview.png)

##### Propiedades para cada bang
| Nombre | Tipo | Requerido |
| ---- | ---- | -------- |
| title | string | no |
| shortcut | string | sí |
| url | string | sí |

###### `title`
Título opcional que aparecerá en el lado derecho de la barra de búsqueda cuando la consulta empiece con el atajo asociado.

###### `shortcut`
Cualquier valor que desees usar como atajo para el motor de búsqueda. No tiene que empezar con `!`.

> [!IMPORTANTE]
>
> En YAML algunos caracteres tienen un significado especial cuando se colocan al principio de un valor. Si tu atajo empieza con `!` (y potencialmente algunos otros caracteres especiales) tendrás que envolver el valor entre comillas:
> ```yaml
> shortcut: "!yt"
>```

###### `url`
La URL del motor de búsqueda. Usa `{QUERY}` para indicar dónde se coloca el valor de la consulta. Ejemplos:

```yaml
url: https://www.reddit.com/search?q={QUERY}
url: https://store.steampowered.com/search/?term={QUERY}
url: https://www.amazon.com/s?k={QUERY}
```

### Grupo
Agrupa múltiples widgets en uno usando pestañas. Los widgets se definen usando una propiedad `widgets` exactamente como lo harías en una columna de página. La única limitación es que no puedes colocar un widget de grupo o un widget de columna dividida dentro de un widget de grupo.

Ejemplo:

```yaml
- type: group
  widgets:
    - type: reddit
      subreddit: gamingnews
      show-thumbnails: true
      collapse-after: 6
    - type: reddit
      subreddit: games
    - type: reddit
      subreddit: pcgaming
      show-thumbnails: true
```

Vista previa:

![](images/group-widget-preview.png)

#### Compartir propiedades

Para evitar la repetición, puedes usar [anclas YAML](https://support.atlassian.com/bitbucket-cloud/docs/yaml-anchors/) y compartir propiedades entre widgets.

Ejemplo:

```yaml
- type: group
  define: &shared-properties
      type: reddit
      show-thumbnails: true
      collapse-after: 6
  widgets:
    - subreddit: gamingnews
      <<: *shared-properties
    - subreddit: games
      <<: *shared-properties
    - subreddit: pcgaming
      <<: *shared-properties
```

### Columna Dividida
Divide una columna de tamaño completo por la mitad, permitiéndote colocar widgets uno al lado del otro horizontalmente. Esto se convierte en una sola columna en dispositivos móviles o si no hay suficiente ancho disponible. Los widgets se definen usando una propiedad `widgets` exactamente como lo harías en una columna de página.

Dos widgets uno al lado del otro en una columna `full`:

![](images/split-column-widget-preview.png)

<details>
<summary>Ver <code>glance.yml</code></summary>
<br>

```yaml
# ...
- size: full
  widgets:
    - type: split-column
      widgets:
        - type: hacker-news
          collapse-after: 3
        - type: lobsters
          collapse-after: 3

    - type: videos
# ...
```
</details>
<br>

También puedes lograr un número de diseños de página completa diferentes usando solo este widget, como:

Diseño de 3 columnas donde todas las columnas tienen el mismo ancho:

![](images/split-column-widget-3-columns.png)

<details>
<summary>Ver <code>glance.yml</code></summary>
<br>

```yaml
pages:
  - name: Home
    columns:
      - size: full
        widgets:
          - type: split-column
            max-columns: 3
            widgets:
              - type: reddit
                subreddit: selfhosted
                collapse-after: 15
              - type: reddit
                subreddit: homelab
                collapse-after: 15
              - type: reddit
                subreddit: sysadmin
                collapse-after: 15
```
</details>
<br>

Diseño de 4 columnas donde todas las columnas tienen el mismo ancho (y la página está establecida en `width: wide`):

![](images/split-column-widget-4-columns.png)

<details>
<summary>Ver <code>glance.yml</code></summary>
<br>

```yaml
pages:
  - name: Home
    width: wide
    columns:
      - size: full
        widgets:
          - type: split-column
            max-columns: 4
            widgets:
              - type: reddit
                subreddit: selfhosted
                collapse-after: 15
              - type: reddit
                subreddit: homelab
                collapse-after: 15
              - type: reddit
                subreddit: linux
                collapse-after: 15
              - type: reddit
                subreddit: sysadmin
                collapse-after: 15
```
</details>
<br>

Diseño de mampostería con hasta 5 columnas donde todas las columnas tienen el mismo ancho (y la página está establecida en `width: wide`):

![](images/split-column-widget-masonry.png)

<details>
<summary>Ver <code>glance.yml</code></summary>
<br>

```yaml
define:
  - &subreddit-settings
    type: reddit
    collapse-after: 5

pages:
  - name: Home
    width: wide
    columns:
      - size: full
        widgets:
          - type: split-column
            max-columns: 5
            widgets:
              - subreddit: selfhosted
                <<: *subreddit-settings
              - subreddit: homelab
                <<: *subreddit-settings
              - subreddit: linux
                <<: *subreddit-settings
              - subreddit: sysadmin
                <<: *subreddit-settings
              - subreddit: DevOps
                <<: *subreddit-settings
              - subreddit: Networking
                <<: *subreddit-settings
              - subreddit: DataHoarding
                <<: *subreddit-settings
              - subreddit: OpenSource
                <<: *subreddit-settings
              - subreddit: Privacy
                <<: *subreddit-settings
              - subreddit: FreeSoftware
                <<: *subreddit-settings
```
</details>
<br>

Al igual que el widget `group`, puedes insertar cualquier tipo de widget, incluso puedes insertar un widget `group` dentro de un widget `split-column`, pero no puedes insertar un widget `split-column` dentro de un widget `group`.


### API Personalizada

Muestra datos de una API JSON usando una plantilla personalizada.

> [!NOTA]
>
> La configuración de este widget requiere algunos conocimientos básicos de programación, HTML, CSS, el lenguaje de plantillas Go y conceptos específicos de Glance.

Ejemplos:

![](images/custom-api-preview-1.png)

<details>
<summary>Ver <code>glance.yml</code></summary>
<br>

```yaml
- type: custom-api
  title: Dato Aleatorio
  cache: 6h
  url: https://uselessfacts.jsph.pl/api/v2/facts/random
  template: |
    <p class="size-h4 color-paragraph">{{ .JSON.String "text" }}</p>
```
</details>
<br>

![](images/custom-api-preview-2.png)

<details>
<summary>Ver <code>glance.yml</code></summary>
<br>

```yaml
- type: custom-api
  title: Estadísticas de Immich
  cache: 1d
  url: https://${IMMICH_URL}/api/server/statistics
  headers:
    x-api-key: ${IMMICH_API_KEY}
    Accept: application/json
  template: |
    <div class="flex justify-between text-center">
      <div>
          <div class="color-highlight size-h3">{{ .JSON.Int "photos" | formatNumber }}</div>
          <div class="size-h6">FOTOS</div>
      </div>
      <div>
          <div class="color-highlight size-h3">{{ .JSON.Int "videos" | formatNumber }}</div>
          <div class="size-h6">VÍDEOS</div>
      </div>
      <div>
          <div class="color-highlight size-h3">{{ div (.JSON.Int "usage" | toFloat) 1073741824 | toInt | formatNumber }}GB</div>
          <div class="size-h6">USO</div>
      </div>
    </div>
```
</details>
<br>

![](images/custom-api-preview-3.png)

<details>
<summary>Ver <code>glance.yml</code></summary>
<br>

```yaml
- type: custom-api
  title: Ofertas Especiales de Steam
  cache: 12h
  url: https://store.steampowered.com/api/featuredcategories?cc=us
  template: |
    <ul class="list list-gap-10 collapsible-container" data-collapse-after="5">
    {{ range .JSON.Array "specials.items" }}
      <li>
        <a class="size-h4 color-highlight block text-truncate" href="https://store.steampowered.com/app/{{ .Int "id" }}/">{{ .String "name" }}</a>
        <ul class="list-horizontal-text">
          <li>{{ div (.Int "final_price" | toFloat) 100 | printf "$%.2f" }}</li>
          {{ $discount := .Int "discount_percent" }}
          <li{{ if ge $discount 40 }} class="color-positive"{{ end }}>{{ $discount }}% de descuento</li>
        </ul>
      </li>
    {{ end }}
    </ul>
```
</details>

#### Propiedades
| Nombre | Tipo | Requerido | Predeterminado |
| ---- | ---- | -------- | ------- |
| url | string | sí | |
| headers | key (string) & value (string) | no | |
| frameless | boolean | no | false |
| template | string | sí | |

##### `url`
La URL de la que se deben obtener los datos. Debe ser accesible desde el servidor en el que se está ejecutando Glance.

##### `headers`
Opcionalmente, especifica los encabezados que se enviarán con la petición. Ejemplo:

```yaml
headers:
  x-api-key: tu-api-key
  Accept: application/json
```

##### `frameless`
Cuando se establece en `true`, elimina el borde y el relleno alrededor del widget.

##### `template`
La plantilla que se usará para mostrar los datos. Se basa en el paquete `html/template` de Go, por lo que se recomienda revisar [su documentación](https://pkg.go.dev/text/template) para entender cómo hacer cosas básicas como condicionales, bucles, etc. Además, también usa el paquete [gjson de tidwall](https://github.com/tidwall/gjson) para analizar los datos JSON, por lo que vale la pena revisar su documentación si quieres usar selectores JSON más avanzados. Puedes ver ejemplos adicionales con explicaciones y definiciones de funciones [aquí](custom-api.md).

### Extensión
Muestra un widget proporcionado por una fuente externa (terceros). Si quieres aprender más sobre el desarrollo de extensiones, consulta la [documentación de extensiones](extensions.md) (WIP).

```yaml
- type: extension
  url: https://domain.com/widget/display-a-message
  allow-potentially-dangerous-html: true
  parameters:
    message: ¡Hola, mundo!
```

#### Propiedades
| Nombre | Tipo | Requerido | Predeterminado |
| ---- | ---- | -------- | ------- |
| url | string | sí | |
| fallback-content-type | string | no | |
| allow-potentially-dangerous-html | boolean | no | false |
| parameters | key & value | no | |

##### `url`
La URL de la extensión. **Ten en cuenta que la consulta se elimina de esta URL y se usa la definida por `parameters` en su lugar.**

##### `fallback-content-type`
Opcionalmente, especifica el tipo de contenido de fallback de la extensión si la URL no devuelve un encabezado `Widget-Content-Type` válido. Actualmente, el único valor soportado para esta propiedad es `html`.

##### `allow-potentially-dangerous-html`
Si se permite que la extensión muestre HTML.

> [!ADVERTENCIA]
>
> Hay una razón por la que esta propiedad suena aterradora. Está destinada a ser usada por desarrolladores que se sienten cómodos desarrollando y usando sus propias extensiones. No la habilites si no tienes ni idea de lo que significa o si no estás **absolutamente seguro** de que la URL de extensión que estás usando es segura.

##### `parameters`
Una lista de claves y valores que se enviarán a la extensión como parámetros de consulta.

### Clima
Muestra información meteorológica para una ubicación específica. Los datos son proporcionados por https://open-meteo.com/.

Ejemplo:

```yaml
- type: weather
  units: metric
  hour-format: 12h
  location: London, United Kingdom
```

> [!NOTA]
>
> Las ciudades de EE.UU. que tienen nombres comunes pueden tener su estado especificado como el segundo parámetro de esta manera:
>
> * Greenville, North Carolina, United States
> * Greenville, South Carolina, United States
> * Greenville, Mississippi, United States


Vista previa:

![](images/weather-widget-preview.png)

Cada barra representa un intervalo de 2 horas. El fondo amarillo representa el amanecer y el atardecer. Los puntos azules representan las horas del día donde hay una alta probabilidad de precipitación. Puedes pasar el ratón por encima de las barras para ver la temperatura exacta para ese momento.

#### Propiedades

| Nombre | Tipo | Requerido | Predeterminado |
| ---- | ---- | -------- | ------- |
| location | string | sí |  |
| units | string | no | metric |
| hour-format | string | no | 12h |
| hide-location | boolean | no | false |
| show-area-name | boolean | no | false |

##### `location`
El nombre de la ciudad y el país para los que se debe obtener información meteorológica. Intentar iniciar la aplicación con una ubicación inválida resultará en un error. Puedes usar la [página de la API de geocodificación](https://open-meteo.com/en/docs/geocoding-api) para buscar tu ubicación específica. Glance usará el primer resultado de la lista si hay varios.

##### `units`
Si se debe mostrar la temperatura en grados Celsius o Fahrenheit, los valores posibles son `metric` o `imperial`.

#### `hour-format`
Si se deben mostrar las horas del día en formato de 12 horas o en formato de 24 horas. Los valores posibles son `12h` y `24h`.

##### `hide-location`
Opcionalmente, no muestra el nombre de la ubicación en el widget.

##### `show-area-name`
Si se debe mostrar el estado/área administrativa en el nombre de la ubicación. Si se establece en `true`, la ubicación se mostrará como:

```
Greenville, North Carolina, United States
```

De lo contrario, si se establece en `false` (que es el valor predeterminado), se mostrará como:

```
Greenville, United States
```

### Monitor
Muestra una lista de sitios y si son accesibles (en línea) o no. Esto se determina enviando una petición GET a la URL especificada, si la respuesta es 200 entonces el sitio está OK. El tiempo que tardó en recibir una respuesta también se muestra en milisegundos.

Ejemplo:

```yaml
- type: monitor
  cache: 1m
  title: Servicios
  sites:
    - title: Jellyfin
      url: https://jellyfin.yourdomain.com
      icon: /assets/jellyfin-logo.png
    - title: Gitea
      url: https://gitea.yourdomain.com
      icon: /assets/gitea-logo.png
    - title: Immich
      url: https://immich.yourdomain.com
      icon: /assets/immich-logo.png
    - title: AdGuard Home
      url: https://adguard.yourdomain.com
      icon: /assets/adguard-logo.png
    - title: Vaultwarden
      url: https://vault.yourdomain.com
      icon: /assets/vaultwarden-logo.png

```

Vista previa:

![](images/monitor-widget-preview.png)

Puedes pasar el ratón por encima del texto "ERROR" para ver más información.

#### Propiedades

| Nombre | Tipo | Requerido | Predeterminado |
| ---- | ---- | -------- | ------- |
| sites | array | sí | |
| style | string | no | |
| show-failing-only | boolean | no | false |

##### `show-failing-only`
Muestra solo una lista de sitios fallidos cuando se establece en `true`.

##### `style`
Usado para cambiar la apariencia del widget. Los valores posibles son `compact`.

Vista previa de `compact`:

![](images/monitor-widget-compact-preview.png)

##### `sites`

Propiedades para cada sitio:

| Nombre | Tipo | Requerido | Predeterminado |
| ---- | ---- | -------- | ------- |
| title | string | sí | |
| url | string | sí | |
| check-url | string | no | |
| error-url | string | no | |
| icon | string | no | |
| allow-insecure | boolean | no | false |
| same-tab | boolean | no | false |
| alt-status-codes | array | no | |

`title`

El título usado para indicar el sitio.

`url`

La URL pública de un servicio monitorizado, el usuario será redirigido aquí. Si no se especifica `check-url`, esto se usa como la comprobación de estado.

`check-url`

La URL que se solicitará y su respuesta determinará el estado del sitio. Si no se especifica, se usa la propiedad `url`.

`error-url`

Si el servicio monitorizado devuelve un error, el usuario será redirigido aquí. Si no se especifica, se usa la propiedad `url`.

`icon`

URL opcional a una imagen que se usará como icono para el sitio. Puede ser una URL externa o interna a través de [assets configurados del servidor](#assets-path). También puedes usar directamente [Simple Icons](https://simpleicons.org/) a través de un prefijo `si:` o [Dashboard Icons](https://github.com/walkxcode/dashboard-icons) a través de un prefijo `di:`:

```yaml
icon: si:jellyfin
icon: si:gitea
icon: si:adguard
```

> [!ADVERTENCIA]
>
> Simple Icons se cargan externamente y están alojados en `cdn.jsdelivr.net`, si no deseas depender de un tercero, eres libre de descargar los iconos individualmente y alojarlos localmente.

`allow-insecure`

Si se deben ignorar los certificados inválidos/autofirmados.

`same-tab`

Si se debe abrir el enlace en la misma pestaña o en una nueva.

`alt-status-codes`

Códigos de estado distintos de 200 que quieres que devuelvan "OK".

```yaml
alt-status-codes:
  - 403
```

### Lanzamientos
Muestra una lista de los últimos lanzamientos para repositorios específicos en Github, GitLab, Codeberg o Docker Hub.

Ejemplo:

```yaml
- type: releases
  show-source-icon: true
  repositories:
    - go-gitea/gitea
    - jellyfin/jellyfin
    - glanceapp/glance
    - codeberg:redict/redict
    - gitlab:fdroid/fdroidclient
    - dockerhub:gotify/server
```

Vista previa:

![](images/releases-widget-preview.png)

#### Propiedades

| Nombre | Tipo | Requerido | Predeterminado |
| ---- | ---- | -------- | ------- |
| repositories | array | sí |  |
| show-source-icon | boolean | no | false |  |
| token | string | no | |
| gitlab-token | string | no | |
| limit | integer | no | 10 |
| collapse-after | integer | no | 5 |

##### `repositories`
Una lista de repositorios para los que se debe obtener el último lanzamiento. Solo se requiere el nombre/repo, no la URL completa. Se puede especificar un prefijo para los repositorios alojados en otro lugar como GitLab, Codeberg y Docker Hub. Ejemplo:

```yaml
repositories:
  - gitlab:inkscape/inkscape
  - dockerhub:glanceapp/glance
  - codeberg:redict/redict
```

Las imágenes oficiales en Docker Hub se pueden especificar omitiendo el propietario:

```yaml
repositories:
  - dockerhub:nginx
  - dockerhub:node
  - dockerhub:alpine
```

También puedes especificar etiquetas exactas para las imágenes de Docker Hub:

```yaml
repositories:
  - dockerhub:nginx:latest
  - dockerhub:nginx:stable-alpine
```

Para incluir pre-lanzamientos, puedes especificar el repositorio como un objeto y usar la propiedad `include-prereleases`:

**Nota: Esta función solo está disponible actualmente para repositorios de GitHub.**

```yaml
repositories:
  - gitlab:inkscape/inkscape
  - repository: glanceapp/glance
    include-prereleases: true
  - codeberg:redict/redict
```

##### `show-source-icon`
Muestra un icono de la fuente (GitHub/GitLab/Codeberg/Docker Hub) junto al nombre del repositorio cuando se establece en `true`.

##### `token`
Sin autenticación, Github permite hasta 60 peticiones por hora. Puedes superar fácilmente este límite y empezar a ver errores si estás rastreando muchos repositorios o tu tiempo de caché es bajo. Para evitar esto, puedes [crear un token de solo lectura desde tu cuenta de Github](https://github.com/settings/personal-access-tokens/new) y proporcionarlo aquí.

También puedes especificar el valor para este token a través de una variable ENV usando la sintaxis `${GITHUB_TOKEN}` donde `GITHUB_TOKEN` es el nombre de la variable que contiene el token. Si has instalado Glance a través de docker, puedes especificar el token en tu docker-compose:

```yaml
services:
  glance:
    image: glanceapp/glance
    environment:
      - GITHUB_TOKEN=<tu token>
```

y luego usarlo en tu `glance.yml` así:

```yaml
- type: releases
  token: ${GITHUB_TOKEN}
  repositories: ...
```

De esta manera, puedes comprobar de forma segura tu `glance.yml` en el control de versiones sin exponer el token.

##### `gitlab-token`
Igual que el anterior pero usado al obtener lanzamientos de GitLab.

##### `limit`
El número máximo de lanzamientos a mostrar.

#### `collapse-after`
Cuántos lanzamientos son visibles antes de que aparezca el botón "MOSTRAR MÁS". Establecer a `-1` para que nunca se colapse.

### Contenedores Docker

Muestra el estado de tus contenedores Docker junto con un icono y una descripción corta opcional.

![](images/docker-containers-preview.png)

```yaml
- type: docker-containers
  hide-by-default: false
```

> [!NOTA]
>
> El widget requiere acceso a `docker.sock`. Si estás ejecutando Glance dentro de un contenedor, esto se puede hacer montando el socket como un volumen:
>
> ```yaml
> services:
>   glance:
>     image: glanceapp/glance
>     volumes:
>       - /var/run/docker.sock:/var/run/docker.sock
> ```

La configuración de los contenedores se hace a través de etiquetas aplicadas a cada contenedor:

```yaml
  jellyfin:
    image: jellyfin/jellyfin:latest
    labels:
      glance.name: Jellyfin
      glance.icon: si:jellyfin
      glance.url: https://jellyfin.domain.com
      glance.description: Películas y series
```

Para servicios con múltiples contenedores, puedes especificar un `glance.id` en el contenedor "principal" y `glance.parent` en cada contenedor "hijo":

<details>
<summary>Ver <code>docker-compose.yml</code></summary>
<br>

```yaml
services:
  immich-server:
    image: ghcr.io/immich-app/immich-server
    labels:
      glance.name: Immich
      glance.icon: si:immich
      glance.url: https://immich.domain.com
      glance.description: Gestión de imágenes y vídeos
      glance.id: immich

  redis:
    image: docker.io/redis:6.2-alpine
    labels:
      glance.parent: immich
      glance.name: Redis

  database:
    image: docker.io/tensorchord/pgvecto-rs:pg14-v0.2.0
    labels:
      glance.parent: immich
      glance.name: DB

  proxy:
    image: nginx:stable
    labels:
      glance.parent: immich
      glance.name: Proxy
```
</details>
<br>

Esto colocará todos los contenedores hijos bajo el contenedor `Immich` al pasar el ratón por encima de su icono:

![](images/docker-container-parent.png)

Si alguno de los contenedores hijos está caído, su estado se propagará al contenedor padre:

![](images/docker-container-parent2.png)

#### Propiedades

| Nombre | Tipo | Requerido | Predeterminado |
| ---- | ---- | -------- | ------- |
| hide-by-default | boolean | no | false |
| sock-path | string | no | /var/run/docker.sock |

##### `hide-by-default`
Si se deben ocultar los contenedores por defecto. Si se establece en `true`, tendrás que añadir manualmente una etiqueta `glance.hide: false` a cada contenedor que quieras mostrar. Por defecto, se mostrarán todos los contenedores y si quieres ocultar un contenedor específico, puedes añadir una etiqueta `glance.hide: true`.

##### `sock-path`
La ruta al socket de Docker.

#### Etiquetas
| Nombre | Descripción |
| ---- | ----------- |
| glance.name | El nombre mostrado en la UI. Si no se especifica, se usará el nombre del contenedor. |
| glance.icon | El icono mostrado en la UI. Puede ser una URL externa o un icono prefijado con si:, sh: o di: como con los widgets de marcadores y monitor |
| glance.url | La URL a la que se redirigirá al usuario al hacer clic en el contenedor. |
| glance.same-tab | Si se debe abrir el enlace en la misma pestaña o en una nueva. El valor predeterminado es `false`. |
| glance.description | Una descripción corta mostrada en la UI. El valor predeterminado está vacío. |
| glance.hide | Si se debe ocultar el contenedor. Si se establece en `true`, el contenedor no se mostrará. El valor predeterminado es `false`. |
| glance.id | El ID personalizado del contenedor. Usado para agrupar contenedores bajo un solo padre. |
| glance.parent | El ID del contenedor padre. Usado para agrupar contenedores bajo un solo padre. |

### Estadísticas DNS
Muestra estadísticas de un resolvedor DNS de bloqueo de anuncios auto-alojado como AdGuard Home o Pi-hole.

Ejemplo:

```yaml
- type: dns-stats
  service: adguard
  url: https://adguard.domain.com/
  username: admin
  password: ${ADGUARD_PASSWORD}
```

Vista previa:

![](images/dns-stats-widget-preview.png)

> [!NOTA]
>
> Al usar AdGuard Home, la tercera estadística en la parte superior será la latencia media y al usar Pi-hole será el número total de dominios bloqueados de todas las listas de anuncios.

#### Propiedades

| Nombre | Tipo | Requerido | Predeterminado |
| ---- | ---- | -------- | ------- |
| service | string | no | pihole |
| allow-insecure | bool | no | false |
| url | string | sí |  |
| username | string | when service is `adguard` |  |
| password | string | when service is `adguard` or `pihole-v6` |  |
| token | string | when service is `pihole` |  |
| hide-graph | bool | no | false |
| hide-top-domains | bool | no | false |
| hour-format | string | no | 12h |

##### `service`
O bien `adguard`, o `pihole` (versión principal 5 y anteriores) o `pihole-v6` (versión principal 6 y posteriores).

##### `allow-insecure`
Si se deben permitir certificados inválidos/autofirmados al hacer la petición al servicio.

##### `url`
La URL base del servicio.

##### `username`
Solo requerido cuando se usa AdGuard Home. El nombre de usuario usado para iniciar sesión en el dashboard de administración.

##### `password`
Requerido cuando se usa AdGuard Home, donde la contraseña es la que se usa para iniciar sesión en el dashboard de administración.

También requerido cuando se usa Pi-hole versión principal 6 y posteriores, donde la contraseña es la que se usa para iniciar sesión en el dashboard de administración o la contraseña de la aplicación, que se puede encontrar en `Settings -> Web Interface / API -> Configure app password`.

##### `token`
Solo requerido cuando se usa Pi-hole versión principal 5 o anteriores. El token API que se puede encontrar en `Settings -> API -> Show API token`.

##### `hide-graph`
Si se debe ocultar el gráfico que muestra el número de consultas a lo largo del tiempo.

##### `hide-top-domains`
Si se debe ocultar la lista de los dominios más bloqueados.

##### `hour-format`
Si se debe mostrar el tiempo relativo en el gráfico en formato `12h` o `24h`.

### Estadísticas del Servidor
Muestra estadísticas como el uso de CPU, el uso de memoria y el uso de disco del servidor en el que se está ejecutando Glance u otros servidores.

Ejemplo:

```yaml
- type: server-stats
  servers:
    - type: local
      name: Servicios
```

Vista previa:

![](images/server-stats-preview.gif)

> [!NOTA]
>
> Este widget está actualmente en desarrollo, algunas funciones podrían no funcionar como se espera o podrían cambiar.

Para mostrar datos de un servidor remoto, necesitas tener el Agente de Glance ejecutándose en ese servidor. Puedes descargar el agente desde [aquí](https://github.com/glanceapp/agent), aunque ten en cuenta que todavía está en desarrollo y podría no funcionar como se espera. En el futuro se añadirá soporte para otros proveedores como Glances.

En caso de que la temperatura de la CPU supere los 80°C, aparecerá un icono de llama junto a la CPU. Los indicadores de progreso también se volverán rojos (o el equivalente a tu color negativo) para, con suerte, llamar tu atención si algo es inusualmente alto:

![](images/server-stats-flame-icon.png)

#### Propiedades
| Nombre | Tipo | Requerido | Predeterminado |
| ---- | ---- | -------- | ------- |
| servers | array | no |  |

##### `servers`
Si no se proporciona, mostrará las estadísticas del servidor en el que se está ejecutando Glance.

##### Propiedades para servidores `local` y `remote`
| Nombre | Tipo | Requerido | Predeterminado |
| ---- | ---- | -------- | ------- |
| type | string | sí |  |
| name | string | no |  |
| hide-swap | boolean | no | false |

###### `type`
Si se deben mostrar estadísticas para el servidor local o un servidor remoto. Los valores posibles son `local` y `remote`.

###### `name`
El nombre del servidor que se mostrará en el widget. Si no se proporciona, se establecerá por defecto al nombre de host del servidor.

###### `hide-swap`
Si se debe ocultar el uso de swap.

##### Propiedades para el servidor `local`
| Nombre | Tipo | Requerido | Predeterminado |
| ---- | ---- | -------- | ------- |
| cpu-temp-sensor | string | no |  |
| hide-mointpoints-by-default | boolean | no | false |
| mountpoints | map\[string\]object | no |  |

###### `cpu-temp-sensor`
El nombre del sensor a usar para la temperatura de la CPU. Cuando no se proporciona, el widget intentará encontrar el correcto, si no lo consigue, la temperatura no se mostrará. Para ver los sensores disponibles puedes usar el comando `sensors`.

###### `hide-mountpoints-by-default`
Si se establece en `true`, tendrás que hacer visible manualmente cada punto de montaje añadiéndole una propiedad `hide: false` así:

```yaml
- type: server-stats
  servers:
    - type: local
      hide-mountpoints-by-default: true
      mountpoints:
        "/":
          hide: false
        "/mnt/data":
          hide: false
```

Esto es útil si estás ejecutando Glance dentro de un contenedor que normalmente monta muchos sistemas de archivos irrelevantes.

###### `mountpoints`
Un mapa de puntos de montaje para mostrar el uso de disco. La clave es la ruta al punto de montaje y el valor es un objeto con propiedades opcionales. Ejemplo:

```yaml
mountpoints:
  "/":
    name: Root
  "/mnt/data":
    name: Data
  "/boot/efi":
    hide: true
```

##### Propiedades para cada `mountpoint`
| Nombre | Tipo | Requerido | Predeterminado |
| ---- | ---- | -------- | ------- |
| name | string | no |  |
| hide | boolean | no | false |

###### `name`
El nombre del punto de montaje que se mostrará en el widget. Si no se proporciona, se establecerá por defecto la ruta del punto de montaje.

###### `hide`
Si se debe ocultar este punto de montaje del widget.

##### Propiedades para servidores `remote`
| Nombre | Tipo | Requerido | Predeterminado |
| ---- | ---- | -------- | ------- |
| url | string | sí |  |
| token | string | no |  |
| timeout | string | no | 3s |

###### `url`
La URL y el puerto del servidor del que se deben obtener las estadísticas.

###### `token`
El token de autenticación a usar al obtener las estadísticas.

###### `timeout`
El tiempo máximo de espera para una respuesta del servidor. El valor es una cadena y debe ser un número seguido de una de las letras s, m, h, d. Ejemplo: `10s` para 10 segundos, `1m` para 1 minuto, etc.

### Repositorio
Muestra información general sobre un repositorio, así como una lista de las últimas pull requests e issues abiertas.

Ejemplo:

```yaml
- type: repository
  repository: glanceapp/glance
  pull-requests-limit: 5
  issues-limit: 3
  commits-limit: 3
```

Vista previa:

![](images/repository-preview.png)

#### Propiedades

| Nombre | Tipo | Requerido | Predeterminado |
| ---- | ---- | -------- | ------- |
| repository | string | sí |  |
| token | string | no | |
| pull-requests-limit | integer | no | 3 |
| issues-limit | integer | no | 3 |
| commits-limit | integer | no | -1 |

##### `repository`
El propietario y el nombre del repositorio del que se mostrará información.

##### `token`
Sin autenticación, Github permite hasta 60 peticiones por hora. Puedes superar fácilmente este límite y empezar a ver errores si tu tiempo de caché es bajo o tienes muchas instancias de este widget. Para evitar esto, puedes [crear un token de solo lectura desde tu cuenta de Github](https://github.com/settings/personal-access-tokens/new) y proporcionarlo aquí.

##### `pull-requests-limit`
El número máximo de últimas pull requests abiertas a mostrar. Establecer a `-1` para no mostrar ninguna.

##### `issues-limit`
El número máximo de últimos issues abiertos a mostrar. Establecer a `-1` para no mostrar ninguno.

##### `commits-limit`
El número máximo de últimos commits a mostrar desde la rama predeterminada. Establecer a `-1` para no mostrar ninguno.

### Marcadores
Muestra una lista de enlaces que se pueden agrupar.

Ejemplo:

```yaml
- type: bookmarks
  groups:
    - links:
        - title: Gmail
          url: https://mail.google.com/mail/u/0/
        - title: Amazon
          url: https://www.amazon.com/
        - title: Github
          url: https://github.com/
        - title: Wikipedia
          url: https://en.wikipedia.org/
    - title: Entretenimiento
      color: 10 70 50
      links:
        - title: Netflix
          url: https://www.netflix.com/
        - title: Disney+
          url: https://www.disneyplus.com/
        - title: YouTube
          url: https://www.youtube.com/
        - title: Prime Video
          url: https://www.primevideo.com/
    - title: Social
      color: 200 50 50
      links:
        - title: Reddit
          url: https://www.reddit.com/
        - title: Twitter
          url: https://twitter.com/
        - title: Instagram
          url: https://www.instagram.com/
```

Vista previa:

![](images/bookmarks-widget-preview.png)


#### Propiedades

| Nombre | Tipo | Requerido |
| ---- | ---- | -------- |
| groups | array | sí |

##### `groups`
Un array de grupos que opcionalmente pueden tener un título y un color personalizado.

###### Propiedades para cada grupo
| Nombre | Tipo | Requerido | Predeterminado |
| ---- | ---- | -------- | ------- |
| title | string | no | |
| color | HSL | no | el color primario del tema |
| links | array | sí | |
| same-tab | boolean | no | false |
| hide-arrow | boolean | no | false |
| target | string | no | |

> [!CONSEJO]
>
> Puedes establecer `same-tab`, `hide-arrow` y `target` o bien en el grupo, lo que los aplicará a todos los enlaces de ese grupo, o bien en cada enlace individual, lo que anulará el valor establecido en el grupo.

###### Propiedades para cada enlace
| Nombre | Tipo | Requerido | Predeterminado |
| ---- | ---- | -------- | ------- |
| title | string | sí | |
| url | string | sí | |
| icon | string | no | |
| same-tab | boolean | no | false |
| hide-arrow | boolean | no | false |
| target | string | no | |

`icon`

URL que apunta a una imagen. También puedes usar directamente [Simple Icons](https://simpleicons.org/) a través de un prefijo `si:` o [Dashboard Icons](https://github.com/walkxcode/dashboard-icons) a través de un prefijo `di:`:

```yaml
icon: si:gmail
icon: si:youtube
icon: si:reddit
```

> [!ADVERTENCIA]
>
> Simple Icons se cargan externamente y están alojados en `cdn.jsdelivr.net`, si no deseas depender de un tercero, eres libre de descargar los iconos individualmente y alojarlos localmente.

`same-tab`

Si se debe abrir el enlace en la misma pestaña o en una nueva.

`hide-arrow`

Si se debe ocultar la flecha de color en cada enlace.

`target`

Establece un valor personalizado para el atributo `target` del enlace. Los valores posibles son `_blank`, `_self`, `_parent` y `_top`, puedes leer más sobre lo que hacen [aquí](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/a#target). Esta propiedad tiene precedencia sobre `same-tab`.

### ChangeDetection.io
Muestra una lista de vigilancias de changedetection.io.

Ejemplo

```yaml
- type: change-detection
  instance-url: https://changedetection.mydomain.com/
  token: ${CHANGE_DETECTION_TOKEN}
```

Vista previa:

![](images/change-detection-widget-preview.png)

#### Propiedades

| Nombre | Tipo | Requerido | Predeterminado |
| ---- | ---- | -------- | ------- |
| instance-url | string | no | `https://www.changedetection.io` |
| token | string | no |  |
| limit | integer | no | 10 |
| collapse-after | integer | no | 5 |
| watches | array of strings | no |  |

##### `instance-url`
La URL que apunta a tu instancia de `changedetection.io`.

##### `token`
El token de acceso API que se puede encontrar en `SETTINGS > API`. Opcionalmente, puedes especificar esto usando una variable de entorno con la sintaxis `${VARIABLE_NAME}`.

##### `limit`
El número máximo de vigilancias a mostrar.

##### `collapse-after`
Cuántas vigilancias son visibles antes de que aparezca el botón "MOSTRAR MÁS". Establecer a `-1` para que nunca se colapse.

##### `watches`
Por defecto, se mostrarán todas las vigilancias configuradas. Opcionalmente, puedes especificar una lista de UUIDs para las vigilancias específicas que quieres que se listen:

```yaml
  - type: change-detection
    watches:
      - 1abca041-6d4f-4554-aa19-809147f538d3
      - 705ed3e4-ea86-4d25-a064-822a6425be2c
```

### Reloj
Muestra un reloj que muestra la hora y la fecha actuales. Opcionalmente, también muestra la hora en otras zonas horarias.

Ejemplo:

```yaml
- type: clock
  hour-format: 24h
  timezones:
    - timezone: Europe/Paris
      label: París
    - timezone: America/New_York
      label: Nueva York
    - timezone: Asia/Tokyo
      label: Tokio
```

Vista previa:

![](images/clock-widget-preview.png)

#### Propiedades

| Nombre | Tipo | Requerido | Predeterminado |
| ---- | ---- | -------- | ------- |
| hour-format | string | no | 24h |
| timezones | array | no |  |

##### `hour-format`
Si se debe mostrar la hora en formato de 12 o 24 horas. Los valores posibles son `12h` y `24h`.

#### Propiedades para cada zona horaria

| Nombre | Tipo | Requerido | Predeterminado |
| ---- | ---- | -------- | ------- |
| timezone | string | sí | |
| label | string | no | |

##### `timezone`
Un identificador de zona horaria como `Europe/London`, `America/New_York`, etc. La lista completa de identificadores disponibles se puede encontrar [aquí](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones).

##### `label`
Opcionalmente, anula el valor de visualización para la zona horaria a algo más significativo como "Casa", "Trabajo" o cualquier otra cosa.


### Calendario
Muestra un calendario.

Ejemplo:

```yaml
- type: calendar
  first-day-of-week: monday
```

Vista previa:

![](images/calendar-widget-preview.png)

#### Propiedades

| Nombre | Tipo | Requerido | Predeterminado |
| ---- | ---- | -------- | ------- |
| first-day-of-week | string | no | monday |

##### `first-day-of-week`
El día de la semana en que empieza el calendario. Todos los días de la semana están disponibles como valores posibles.

### Calendario (legado)
Muestra un calendario.

Ejemplo:

```yaml
- type: calendar-legacy
  start-sunday: false
```

Vista previa:

![](images/calendar-legacy-widget-preview.png)

> [!NOTA]
>
> Este widget está obsoleto y podría eliminarse en una versión futura.

#### Propiedades

| Nombre | Tipo | Requerido | Predeterminado |
| ---- | ---- | -------- | ------- |
| start-sunday | boolean | no | false |

##### `start-sunday`
Si las semanas del calendario empiezan en domingo o lunes.

> [!NOTA]
>
> Actualmente hay poca personalización disponible para el calendario. Se añadirán funciones extra en el futuro.

### Mercados
Muestra una lista de mercados, su valor actual, el cambio del día y un pequeño gráfico de 21 días. Los datos se toman de Yahoo Finance.

Ejemplo:

```yaml
- type: markets
  markets:
    - symbol: SPY
      name: S&P 500
    - symbol: BTC-USD
      name: Bitcoin
      chart-link: https://www.tradingview.com/chart/?symbol=INDEX:BTCUSD
    - symbol: NVDA
      name: NVIDIA
    - symbol: AAPL
      symbol-link: https://www.google.com/search?tbm=nws&q=apple
      name: Apple
```

Vista previa:

![](images/markets-widget-preview.png)

#### Propiedades

| Nombre | Tipo | Requerido |
| ---- | ---- | -------- |
| markets | array | sí |
| sort-by | string | no |
| chart-link-template | string | no |
| symbol-link-template | string | no |

##### `markets`
Un array de mercados para los que se debe mostrar información.

##### `sort-by`
Por defecto, los mercados se muestran en el orden en que se definieron. Puedes personalizar su ordenación estableciendo la propiedad `sort-by` a `change` para el orden descendente basado en el cambio porcentual de la acción (por ejemplo, 1% se ordenaría más alto que -1%) o `absolute-change` para el orden descendente basado en el cambio de precio absoluto de la acción (por ejemplo, -1% se ordenaría más alto que +0.5%).

##### `chart-link-template`
Una plantilla para el enlace al que ir al hacer clic en el gráfico que se aplicará a todos los mercados. El valor `{SYMBOL}` se reemplazará con el símbolo del mercado. Puedes anular esto por mercado especificando una propiedad `chart-link`. Ejemplo:

```yaml
chart-link-template: https://www.tradingview.com/chart/?symbol={SYMBOL}
```

##### `symbol-link-template`
Una plantilla para el enlace al que ir al hacer clic en el símbolo que se aplicará a todos los mercados. El valor `{SYMBOL}` se reemplazará con el símbolo del mercado. Puedes anular esto por mercado especificando una propiedad `symbol-link`. Ejemplo:

```yaml
symbol-link-template: https://www.google.com/search?tbm=nws&q={SYMBOL}
```

###### Propiedades para cada mercado
| Nombre | Tipo | Requerido |
| ---- | ---- | -------- |
| symbol | string | sí |
| name | string | no |
| symbol-link | string | no |
| chart-link | string | no |

`symbol`

El símbolo, como se ve en Yahoo Finance.

`name`

El nombre que se mostrará bajo el símbolo.

`symbol-link`

El enlace al que ir al hacer clic en el símbolo.

`chart-link`

El enlace al que ir al hacer clic en el gráfico.

### Canales de Twitch
Muestra una lista de canales de Twitch.

Ejemplo:

```yaml
- type: twitch-channels
  channels:
    - jembawls
    - giantwaffle
    - asmongold
    - cohhcarnage
    - j_blow
    - xQc
```

Vista previa:

![](images/twitch-channels-widget-preview.png)

#### Propiedades
| Nombre | Tipo | Requerido | Predeterminado |
| ---- | ---- | -------- | ------- |
| channels | array | sí | |
| collapse-after | integer | no | 5 |
| sort-by | string | no | viewers |

##### `channels`
Una lista de canales a mostrar.

##### `collapse-after`
Cuántos canales son visibles antes de que aparezca el botón "MOSTRAR MÁS". Establecer a `-1` para que nunca se colapse.

##### `sort-by`
Se puede usar para especificar el orden en que se muestran los canales. Los valores posibles son `viewers` y `live`.

### Juegos Top de Twitch
Muestra una lista de juegos con más espectadores en Twitch.

Ejemplo:

```yaml
- type: twitch-top-games
  exclude:
    - just-chatting
    - pools-hot-tubs-and-beaches
    - music
    - art
    - asmr
```

Vista previa:

![](images/twitch-top-games-widget-preview.png)

#### Propiedades
| Nombre | Tipo | Requerido | Predeterminado |
| ---- | ---- | -------- | ------- |
| exclude | array | no | |
| limit | integer | no | 10 |
| collapse-after | integer | no | 5 |

##### `exclude`
Una lista de categorías que nunca se mostrarán. Debes proporcionar el slug que se encuentra al hacer clic en la categoría y mirar la URL:

```
https://www.twitch.tv/directory/category/grand-theft-auto-v
                                         ^^^^^^^^^^^^^^^^^^
```

##### `limit`
El número máximo de juegos a mostrar.

##### `collapse-after`
Cuántos juegos son visibles antes de que aparezca el botón "MOSTRAR MÁS". Establecer a `-1` para que nunca se colapse.

### iframe
Incrusta un iframe como widget.

Ejemplo:

```yaml
- type: iframe
  source: <url>
  height: 400
```

#### Propiedades
| Nombre | Tipo | Requerido | Predeterminado |
| ---- | ---- | -------- | ------- |
| source | string | sí | |
| height | integer | no | 300 |

##### `source`
La fuente del iframe.

##### `height`
La altura del iframe. La altura mínima permitida es 50.

### HTML
Incrusta cualquier HTML.

Ejemplo:

```yaml
- type: html
  source: |
    <p>Hola, <span class="color-primary">Mundo</span>!</p>
```

Ten en cuenta el uso de `|` después de `source:`, esto te permite insertar una cadena multilínea.

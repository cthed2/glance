# Extensiones

> [!IMPORTANTE]
>
> **Este documento, así como la funcionalidad de extensiones, son un trabajo en progreso. La API puede cambiar en el futuro. Eres responsable de mantener tus propias extensiones.**

## Resumen

Con la intención de requerir un conocimiento mínimo para desarrollar extensiones, en lugar de ser un protocolo complicado, no son más que una petición HTTP a un servidor que devuelve algunos encabezados especiales. El intercambio entre Glance y las extensiones se puede ver en el siguiente diagrama:

![](images/extension-overview.png)

Si sabes cómo configurar un servidor HTTP y un poco de HTML y CSS, estás listo para empezar a construir tus propias extensiones.

> [!CONSEJO]
>
> Por defecto, el widget de extensión tiene un tiempo de caché de 30 minutos. Para evitar tener que reiniciar Glance después de cada cambio de extensión, puedes establecer el tiempo de caché del widget a 1 segundo:
> ```yaml
> - type: extension
>   url: http://localhost:8081
>   cache: 1s
> ```

## Encabezados

### `Widget-Title`
Se usa para especificar el título del widget. Si no se proporciona, el título del widget será "Extensión".

### `Widget-Content-Type`
Se usa para especificar el tipo de contenido que será devuelto por la extensión. Si no se proporciona, el contenido se mostrará como texto plano.

### `Widget-Content-Frameless`
Cuando se establece en `true`, el contenido del widget se mostrará sin el fondo o "marco" predeterminado.

## Tipos de Contenido

> [!NOTA]
>
> Actualmente, `html` es el único tipo de contenido soportado. El objetivo a largo plazo es tener tipos de contenido genéricos como `videos`, `forum-posts`, `markets`, `streams`, etc., que se devolverán en formato JSON y Glance los mostrará usando estilos y funcionalidades existentes, permitiendo a los desarrolladores de extensiones lograr un aspecto nativo mientras solo se centran en proporcionar datos de su fuente preferida.

### `html`
Muestra el contenido como HTML. Esto requiere que el usuario tenga la propiedad `allow-potentially-dangerous-html` establecida en `true`, de lo contrario, el contenido se mostrará como texto plano.


#### Usar clases y funcionalidades existentes
La mayoría de las características que se ven en Glance se pueden usar fácilmente en tus extensiones HTML personalizadas. A continuación, se muestra un ejemplo de algunas de estas características:

```html
<p class="color-subdue">Texto con color tenue</p>
<p>Texto con color base</p>
<p class="color-highlight">Texto con color resaltado</p>
<p class="color-primary">Texto con color primario</p>
<p class="color-positive">Texto con color positivo</p>
<p class="color-negative">Texto con color negativo</p>

<hr class="margin-block-15">

<p class="size-h1">Tamaño de fuente 1</p>
<p class="size-h2">Tamaño de fuente 2</p>
<p class="size-h3">Tamaño de fuente 3</p>
<p class="size-h4">Tamaño de fuente 4</p>
<p class="size-base">Tamaño de fuente base</p>
<p class="size-h5">Tamaño de fuente 5</p>
<p class="size-h6">Tamaño de fuente 6</p>

<hr class="margin-block-15">

<a class="visited-indicator" href="#notvisitedprobably">Enlace con indicador de visitado</a>

<hr class="margin-block-15">

<a class="color-primary-if-not-visited" href="#notvisitedprobably">Enlace con color primario si no está visitado</a>

<hr class="margin-block-15">

<p>Evento ocurrido hace <span data-dynamic-relative-time="<marca de tiempo Unix>"></span></p>

<hr class="margin-block-15">

<ul class="list-horizontal-text">
    <li>horizontal</li>
    <li>lista</li>
    <li>con</li>
    <li>múltiples</li>
    <li>elementos</li>
    <li>de texto</li>
</ul>

<hr class="margin-block-15">

<ul class="list list-gap-10 list-with-separator">
    <li>lista</li>
    <li>con</li>
    <li>espacio</li>
    <li>y</li>
    <li>líneas</li>
    <li>horizontales</li>
</ul>

<hr class="margin-block-15">

<ul class="list collapsible-container" data-collapse-after="3">
    <li>lista</li>
    <li>colapsable</li>
    <li>con</li>
    <li>muchos</li>
    <li>elementos</li>
    <li>que</li>
    <li>aparecerán</li>
    <li>cuando</li>
    <li>hagas</li>
    <li>clic</li>
    <li>en el</li>
    <li>botón</li>
    <li>de abajo</li>
</ul>

<hr class="margin-bottom-15">

<p class="margin-bottom-10">Imagen cargada perezosamente:</p>

<img src="https://picsum.photos/200" alt="" loading="lazy">

<hr class="margin-block-15">

<p class="margin-bottom-10">Lista de publicaciones:</p>

<ul class="list list-gap-14 collapsible-container" data-collapse-after="5">
    <li>
        <a class="size-h3 color-primary-if-not-visited" href="#link">Lorem ipsum dolor, sit amet consectetur adipisicing elit. Voluptatum, ipsa?</a>
        <ul class="list-horizontal-text">
            <li data-dynamic-relative-time="<marca de tiempo Unix>"></li>
            <li>3,321 puntos</li>
            <li>139 comentarios</li>
        </ul>
    </li>
    <li>
        <a class="size-h3 color-primary-if-not-visited" href="#link">Lorem ipsum dolor, sit amet consectetur adipisicing elit. Voluptatum, ipsa?</a>
        <ul class="list-horizontal-text">
            <li data-dynamic-relative-time="<marca de tiempo Unix>"></li>
            <li>3,321 puntos</li>
            <li>139 comentarios</li>
        </ul>
    </li>
    <li>
        <a class="size-h3 color-primary-if-not-visited" href="#link">Lorem ipsum dolor, sit amet consectetur adipisicing elit. Voluptatum, ipsa?</a>
        <ul class="list-horizontal-text">
            <li data-dynamic-relative-time="<marca de tiempo Unix>"></li>
            <li>3,321 puntos</li>
            <li>139 comentarios</li>
        </ul>
    </li>
</ul>
```

Todo eso resultará en lo siguiente:

![](images/extension-html-reusing-existing-features-preview.png)

**Los nombres de clase o las funcionalidades pueden cambiar, una vez más, eres responsable de mantener tus propias extensiones.**

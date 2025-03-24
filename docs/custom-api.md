[Saltar a las definiciones de función](#funciones)

## Ejemplos

La mejor manera de hacerse una idea de cómo funcionan las plantillas sería con un montón de ejemplos. Aquí están los casos de uso más comunes:

Respuesta JSON:

```json
{
  "title": "My Title",
  "content": "My Content",
}
```

Para acceder a los dos campos en la respuesta JSON, usarías lo siguiente:

```html
<div>{{ .JSON.String "title" }}</div>
<div>{{ .JSON.String "content" }}</div>
```

Salida:

```html
<div>My Title</div>
<div>My Content</div>
```

<hr>

Respuesta JSON:

```json
{
  "author": "John Doe",
  "posts": [
    {
      "title": "My Title",
      "content": "My Content"
    },
    {
      "title": "My Title 2",
      "content": "My Content 2"
    }
  ]
}
```

Para iterar a través del array de publicaciones, usarías lo siguiente:

```html
{{ range .JSON.Array "posts" }}
  <div>{{ .String "title" }}</div>
  <div>{{ .String "content" }}</div>
{{ end }}
```

Salida:

```html
<div>My Title</div>
<div>My Content</div>
<div>My Title 2</div>
<div>My Content 2</div>
```

Observa la falta de `.JSON` al acceder al título y al contenido, esto se debe a que la función `range` establece el contexto al elemento actual del array.

Si quieres acceder al contexto de nivel superior dentro del `range`, puedes usar lo siguiente:

```html
{{ range .JSON.Array "posts" }}
  <div>{{ .String "title" }}</div>
  <div>{{ .String "content" }}</div>
  <div>{{ $.JSON.String "author" }}</div>
{{ end }}
```

Salida:

```html
<div>My Title</div>
<div>My Content</div>
<div>John Doe</div>
<div>My Title 2</div>
<div>My Content 2</div>
<div>John Doe</div>
```

<hr>

Respuesta JSON:

```json
[
    "Apple",
    "Banana",
    "Cherry",
    "Watermelon"
]
```

De forma un tanto extraña, cuando el contexto actual es un tipo básico que no es un objeto, la forma de especificar su tipo es usar una cadena vacía como clave. Así que, para iterar a través del array de cadenas, usarías lo siguiente:

```html
{{ range .JSON.Array "" }}
  <div>{{ .String "" }}</div>
{{ end }}
```

Salida:

```html
<div>Apple</div>
<div>Banana</div>
<div>Cherry</div>
<div>Watermelon</div>
```

Para acceder a un elemento en un índice específico, podrías usar lo siguiente:

```html
<div>{{ .JSON.String "0" }}</div>
```

Salida:

```html
<div>Apple</div>
```

<hr>

Respuesta JSON:

```json
{
    "user": {
        "address": {
            "city": "New York",
            "state": "NY"
        }
    }
}
```

Para acceder fácilmente a objetos profundamente anidados, puedes usar la siguiente notación de puntos:

```html
<div>{{ .JSON.String "user.address.city" }}</div>
<div>{{ .JSON.String "user.address.state" }}</div>
```

Salida:

```html
<div>New York</div>
<div>NY</div>
```

También se soporta el uso de índices en cualquier parte de la ruta:

```json
{
    "users": [
        {
            "name": "John Doe"
        },
        {
            "name": "Jane Doe"
        }
    ]
}
```

```html
<div>{{ .JSON.String "users.0.name" }}</div>
<div>{{ .JSON.String "users.1.name" }}</div>
```

Salida:

```html
<div>John Doe</div>
<div>Jane Doe</div>
```

<hr>

Respuesta JSON:

```json
{
    "user": {
        "name": "John Doe",
        "age": 30
    }
}
```

Para comprobar si un campo existe, puedes usar lo siguiente:

```html
{{ if .JSON.Exists "user.age" }}
  <div>{{ .JSON.Int "user.age" }}</div>
{{ else }}
  <div>Edad no proporcionada</div>
{{ end }}
```

Salida:

```html
<div>30</div>
```

<hr>

Respuesta JSON:

```json
{
    "price": 100,
    "discount": 10
}
```

Se pueden realizar cálculos, sin embargo, todos los números deben convertirse primero a floats si no lo son ya:

```html
<div>{{ sub (.JSON.Int "price" | toFloat) (.JSON.Int "discount" | toFloat) }}</div>
```

Salida:

```html
<div>90</div>
```

Otras operaciones incluyen `add`, `mul` y `div`.

<hr>

En algunos casos, es posible que quieras conocer el código de estado de la respuesta. Esto se puede hacer usando lo siguiente:

```html
{{ if eq .Response.StatusCode 200 }}
  <p>¡Éxito!</p>
{{ else }}
  <p>Error al obtener datos</p>
{{ end }}
```

También puedes acceder a los encabezados de la respuesta:

```html
<div>{{ .Response.Header.Get "Content-Type" }}</div>
```

## Funciones

Las siguientes funciones están disponibles en el objeto `JSON`:

- `String(key string) string`: Devuelve el valor de la clave como una cadena.
- `Int(key string) int`: Devuelve el valor de la clave como un entero.
- `Float(key string) float`: Devuelve el valor de la clave como un float.
- `Bool(key string) bool`: Devuelve el valor de la clave como un booleano.
- `Array(key string) []JSON`: Devuelve el valor de la clave como un array de objetos `JSON`.
- `Exists(key string) bool`: Devuelve true si la clave existe en el objeto JSON.

Las siguientes funciones de ayuda proporcionadas por Glance están disponibles:

- `toFloat(i int) float`: Convierte un entero a un float.
- `toInt(f float) int`: Convierte un float a un entero.
- `add(a, b float) float`: Suma dos números.
- `sub(a, b float) float`: Resta dos números.
- `mul(a, b float) float`: Multiplica dos números.
- `div(a, b float) float`: Divide dos números.
- `formatApproxNumber(n int) string`: Formatea un número para que sea más legible para humanos, ej. 1000 -> 1k.
- `formatNumber(n float|int) string`: Formatea un número con comas, ej. 1000 -> 1.000.

Las siguientes funciones de ayuda proporcionadas por `text/template` de Go están disponibles:

- `eq(a, b any) bool`: Compara dos valores para igualdad.
- `ne(a, b any) bool`: Compara dos valores para desigualdad.
- `lt(a, b any) bool`: Compara dos valores para menor que.
- `lte(a, b any) bool`: Compara dos valores para menor o igual que.
- `gt(a, b any) bool`: Compara dos valores para mayor que.
- `gte(a, b any) bool`: Compara dos valores para mayor o igual que.
- `and(a, b bool) bool`: Devuelve true si ambos valores son true.
- `or(a, b bool) bool`: Devuelve true si alguno de los valores es true.
- `not(a bool) bool`: Devuelve lo opuesto al valor.
- `index(a any, b int) any`: Devuelve el valor en el índice especificado de un array.
- `len(a any) int`: Devuelve la longitud de un array.
- `printf(format string, a ...any) string`: Devuelve una cadena formateada.

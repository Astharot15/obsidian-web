#web #vuln 

Esta vulnerabilidad consiste en aprovechar el hecho de que una página permita la subida de archivos de algún tipo para la obtención de acceso al servidor. Esta vulnerabilidad se desarrolla por un mal filtrado de archivos subidos o la inexistencia de uno. El objetivo es subir un archivo con una [[Reverse shell]] o un archivo malicioso para que luego se ejecute con una petición conociendo la ruta dónde se guarda o la ejecución por parte del administrador.
Hay diferentes formas de filtrado que utilizan los servicios web.
La que se usa normalmente es el filtrado por tipo de archivo, sin embargo, este tipo de filtrado puede ser de diferentes formas:
	Por el texto que hay a partir del último punto del nombre del archivo. Esto es sencillo de saltar con un ataque de byte nulo. Esto se logra poniendo al final del nombre del archivo %00 para que la página web detecte que después del último punto pone .png pero es ignorado por el null byte por lo que acaba siendo un .php:
	image.php%00.png

## Ejecución de código php

Una página mal configurada puede permitir subir archivos de un tipo malicioso, PHP por ejemplo. Podemos ejecutar código de la siguiente forma:

```php
<?php system("command"); ?>
```

Solo tendríamos que llegar al archivo dentro del servidor para ver el resultado del comando.
Podemos crear una web shell para ejecutar el comando que queramos.

```php
<?php system($_REQUEST['cmd']); ?>
```

A parte de PHP podemos utilizar otras web shells como de archivos .NET

```.NET
<% eval request('cmd') %>
```

Con este archivo subido podemos usar la web shell de la siguiente forma:

```URL
http://web.com/uploads/archivo.php?cmd=command
```

## Bypass client side

En ocasiones, para evitar esta vulnerabilidad se permiten solo archivos de un tipo. Cuando el tipo de archivo se valida en el lado del cliente podemos hacer dos cosas.

Enviar el archivo malicioso con la extensión que acepte para en burpsuite cambiarlo al tipo de archivo que era.
Cambiar la validación con inspeccionar para que acepte el archivo que intentamos meter o quitar la limitación de tipo de archivo.

## Bypass blacklist extensions

Para evitar el ataque a veces crean una blacklist con extensiones maliciosas. Para bypassearlo haremos fuzzing para encontrar una extensión con la misma funcionalidad que esté permitida y la usamos.

## Bypass whitelisted extensions

Puede haber una whitelist en la que solo permita un tipo de extensión cómo puede ser imágenes. Para bypassearlo podemos utilizar dos puntos para matchear la extensión que piden y que además tenga la que queremos, ej `shell.jpg.php`.

Esto pueden evitarlo utilizando regex para decir que después de la extensión no puede haber nada:

```php
if (!preg_match('/^.*\.(jpg|jpeg|png|gif)$/', $fileName)) { ...SNIP... }
```

En el archivo de configuración de apache2 (/etc/apache2/mods-enabled/php7.4.conf) podemos ver que tipos de archivos están permitidos ejecutarse y si no utilizan una expresión regex pueden ejecutarse archivos como `shell.php.jpg` lo que bypassearía la whitelist.

### Null byte

En el caso de que lo anterior no funcionara, tendríamos que buscar una forma de que nuestro archivo acabe en .php pero que pueda bypassear la whitelist y pille el .jpg. Para ello podemos utilizar el byte nulo o similares que consiste en escribir un carácter para que a partir de este cuente como que no hay nada más, pero a la hora de ejecutarlo se pueda. Se haría de esta forma `shell.jpg%00.php`. Otro ejemplos son:
- `%20`
- `%0a`
- `%00`
- `%0d0a`
- `/`
- `.\`
- `.`
- `…`
- `:`
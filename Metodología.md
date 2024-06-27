#guía #cookies

#### Pasos:

1. Observar el funcionamiento de la página en busca de vulnerabilidades comunes.
	Lista con las vulnerabilidades más comunes:
		[[Broken authentication]]
		[[Cross-site Scripting (XSS)]]
		[[SQL injection]]
		[[File upload]]
		[[Command injection]]
		[[Server-side request forgery (SSRF)]]
		[[Cross-site request forgery (CSRF)]]
		[[Local file inclusion (LFI)]]
		[[Remote file inclusion (RFI)]]
		[[Server side attacks]]
1. Inspeccionar para ver el código público y el método de gestión de cookies
	Las cookies tienen diferentes tipos de cifrado. Algunos tienen [[Base64]] o [[AES]], cifrado estándar. Cuando una cookie inicia por ga significa Google Analitics, por lo que se almacena ahí para diferentes procesamientos de información
	Hay editores de cookies como por ejemplo [[Cookie editor]].
3. Abrir el [[Burpsuite]] y observar el formato de las peticiones, cabeceras, cookies, información enviada. Al igual que el formato de las respuestas interceptando de las peticiones [[HTTP]].

Brightspeed
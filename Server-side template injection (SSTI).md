Consiste en la inyección de código en templates, fragmentos de código que se reutiliza para cambiar x parámetros sin modificar el resto.

Si no se sanitiza la entrada y lo introducido directamente se evalúa permitirá inyección de código. El mejor ejemplo sería inyectar `{{7*7}}` que en el caso de ser vulnerable devolverá el resultado de la operación.

Para identificarlo tenemos una serie de payloads:

```html
{7*7}
${7*7}
#{7*7}
%{7*7}
{{7*7}}
...
```

También se puede identificar con caracteres especiales como  `${{<%[%'"}}%\`

Para identificar el motor de plantilla podemos seguir el siguiente diagrama.

![[Pasted image 20240216100244.png]]

{{.}}
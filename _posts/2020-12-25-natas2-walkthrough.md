---
category: natas
title: Natas2 - Directory Indexing Vulnerability
---
website: http://natas2.natas.labs.overthewire.org (Password: ZluruAthQk7Q2MqmDeTiUij2ZvWy2mBi)

This time no password is contained in the page source, but it is embedded with a PNG file:
```html
<img src="files/pixel.png">
```

From the file path, we know that this is a file stored on the server. Typing the link "http://natas2.natas.labs.overthewire.org/files/" in the browser, I realized that directory browsing on this website is not disabled. Going through that directory and I found a file for the next level password: <small>sJIJNW6ucpu6HPZ1ZAchaDtwd7oGrD14</small>.

<strong>CONCLUSION</strong>

This level is about directory browsing vulnerability, which is a type of <em>Security Misconfiguration</em> in OWASP Top Ten. For more details, please refer to links below for more details:
1. http://projects.webappsec.org/w/page/13246922/Directory%20Indexing
2. https://owasp.org/www-project-top-ten/2017/A6_2017-Security_Misconfiguration

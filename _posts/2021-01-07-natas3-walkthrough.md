---
category: natas
title: Natas3 - robots.txt
---
website: http://natas3.natas.labs.overthewire.org (password: sJIJNW6ucpu6HPZ1ZAchaDtwd7oGrD14)

After viewing page source, I found this comment line "Not even Google will find it this time". This is related with something called <em>website crawl</em>. Search engines such as Google can crawl through websites and index on what they find. One way web admins can help to defend against standard web crawlers is to use a file named `robots.txt` at the root of their site. Once a file or a directory is listed in this file, honest search engines and web crawlers will not touch those listed resources. However, it will not prevent hackers to dig into them.

With this knowledge in mind, I found this line <small>Disallow: /s3cr3t/</small> in natas3's robots.txt. Again directory indexing is not disabled in that folder and I found the file for the password.

<strong>CONCLUSION</strong>

Be careful with web crawlers, especially those non-ethical hackers. They can find hidden directories on your website and dig into them. You can find more info about robots.txt <a href="https://www.robotstxt.org/robotstxt.html">here</a>.

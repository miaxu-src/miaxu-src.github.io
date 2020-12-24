---
category: natas
---

In this series, I will dive into <a href="https://overthewire.org/wargames/natas/">Natas</a>, a CTF-like wargame offered by the OverTheWire community. Natas teaches the basics of server-side web application security and covers many of OWASP Top Ten Web Application Security Risks. It consists of different levels, and each level contains the password to the next level. In this wargame, your job is to crack that each password and level up.

Let's start with natas0. According to the hint on that webpage, the password for the next level is on this page. Right click and view page source. There is the password for level 1: <small>gtVrDuiDfck831PqWsLEZy5gyDz1clto</small>.

Once in natas1, we can see that rightclicking is blocked. No worries. We can still view page source from a web browser's menu. Again the password is hardcoded in a comment line: <small>ZluruAthQk7Q2MqmDeTiUij2ZvWy2mBi</small>.

Hardcoding sensitive information is still something we can see occasionally if not all the time. Besides that, sometimes you may also find sensitive information in an application's log files, and even worse, maybe in plaintext! No idea? Just take a look at this article: <a href="https://systemoverlord.com/2018/05/03/how-the-twitter-and-github-password-logging-issues-could-happen.html">How the Twitter and GitHub Password Logging Issues Could Happen</a>.

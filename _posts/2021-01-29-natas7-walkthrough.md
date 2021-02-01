---
category: natas
title: Natas7 Walkthrough - File Inclusion Vulnerability
---

website: http://natas7.natas.labs.overthewire.org/ (password: 7z3hEENjQtflzgnT29q7wAvMNfZdh0i9)

After logging into natas7, I could see this page has two links. Clicking on the links directs me to different pages. For example, <small>http://natas7.natas.labs.overthewire.org/index.php?page=about</small>. This reminds me the <em>File Inclusion Vulnerability</em>, that is, a web application naively builds a file path from a user's input without being sanitized. Moreover, the page source has a comment saying that <small>password for webuser natas8 is in /etc/natas_webpass/natas8</small>. Thus, I tried the following cURL command and it returned me the password for natas8:
```console
curl http://natas7:7z3hEENjQtflzgnT29q7wAvMNfZdh0i9@natas7.natas.labs.overthewire.org?page=/etc/natas_webpass/natas8
```

<strong>CONCLUSION</strong>

File inclusion vulnerability is typically related with PHP functions such as `include()` and `require()`. In these functions, another file will be sourced into the current page for display or execution. If parameters of these functions come from user inputs and are not validated, then an attacker can exploit this vulnerability to include malicious code from remote sources.

<strong>References</strong>

- <a href='https://en.wikipedia.org/wiki/File_inclusion_vulnerability'>https://en.wikipedia.org/wiki/File_inclusion_vulnerability</a>
- <a href='https://www.owasp.org/index.php/Testing_for_Local_File_Inclusion'>https://www.owasp.org/index.php/Testing_for_Local_File_Inclusion</a>

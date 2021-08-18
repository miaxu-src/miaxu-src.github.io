---
category: natas
title: Natas28 - Broken cryptography and SQL injection
---

website: http://natas28.natas.labs.overthewire.org/ (password: JWwR438wkgTsNKBbcJoowyysdM82YjeF)

This is another very challenging level. No source code is provided in this challenge.
In Firefox using the Web Developer Tools, I noticed the web page was redirected to another URL:
- http://natas28.natas.labs.overthewire.org/search.php/?query=G+glEae6W/1XjA7vRm21nNyEco/c+J2TdR0Qp8dcjPKriAqPE2++uYlniRMkobB1vfoQVOxoUVz5bypVRFkZR5BPSyq/LC12hqpypTFRyXA=

Obviously the "query" parameter in the search.php is related with our input, but base64 encoded.
I replaced the "query" parameter with "?query=a", the page now returned an error message:
"Incorrect amount of PKCS#7 padding for blocksize". 
It's a good sign that the user input is encrypted before being sent to the web server.

At this time I didn't have any further clue because no source code was shown in this level.
I asked a friend, who is a real security expert and a hardcore CTF player, for help.
With his help, I came up with a script that feeds the input with different characters and different input lengths:

```bash
#!/bin/bash

USERNAME='natas28'
PASSWORD='JWwR438wkgTsNKBbcJoowyysdM82YjeF'

send_query()
{
    ret=$(curl -isu "$USERNAME:$PASSWORD" "http://natas28.natas.labs.overthewire.org" -F "query=$1" | grep Location | cut -d'=' -f2)
    ret=$(urlencode -d "$ret")
}

# loop through different letters
echo "loop through letters..."
for input in {a..z}; do
    send_query "$input"
    #echo "$ret"
    printf "%2s:$ret\n" "$input"
done

# loop through different input lengths
echo "loop through input lengths..."
for len in {1..50}; do
    input=$(printf 'a%.0s' $(seq 1 $len))
    send_query "$input"
    printf "%2s:$ret\n" "$len"
done
```

Sample output is attached below:
```plain
# bash /tmp/test.sh
loop through letters...
 a:G+glEae6W/1XjA7vRm21nNyEco/c+J2TdR0Qp8dcjPKriAqPE2++uYlniRMkobB1vfoQVOxoUVz5bypVRFkZR5BPSyq/LC12hqpypTFRyXA=
 b:G+glEae6W/1XjA7vRm21nNyEco/c+J2TdR0Qp8dcjPIYiwNnSJY7KHJGU+XjuMzVvfoQVOxoUVz5bypVRFkZR5BPSyq/LC12hqpypTFRyXA=
 c:G+glEae6W/1XjA7vRm21nNyEco/c+J2TdR0Qp8dcjPKEMZKNASy09t5ooTNAbaX0vfoQVOxoUVz5bypVRFkZR5BPSyq/LC12hqpypTFRyXA=
... <skipped>
loop through input lengths...
 1:G+glEae6W/1XjA7vRm21nNyEco/c+J2TdR0Qp8dcjPKriAqPE2++uYlniRMkobB1vfoQVOxoUVz5bypVRFkZR5BPSyq/LC12hqpypTFRyXA=
 2:G+glEae6W/1XjA7vRm21nNyEco/c+J2TdR0Qp8dcjPKxMKUxvsiccFITv6XJZnrHSHmaB7HSm1mCAVyTVcLgDq3tm9uspqc7cbNaAQ0sTFc=
 3:G+glEae6W/1XjA7vRm21nNyEco/c+J2TdR0Qp8dcjPIvUpOmOsuf6Me06CS3bWodmi4rXbbzHxmhT3Vnjq2qkEJJuT5N6gkJR5mVucRLNRo=
... <skipped>
13:G+glEae6W/1XjA7vRm21nNyEco/c+J2TdR0Qp8dcjPLAhy3ui8kLEVaROwiiI6OeH3RxTXb8xdRkxqIh5u2Y5GIjoU2cQpG5h3WwP7xz1O3YrlHX2nGysIPZGaDXuIuY
14:G+glEae6W/1XjA7vRm21nNyEco/c+J2TdR0Qp8dcjPLAhy3ui8kLEVaROwiiI6Oe7NNvj9kWTUA1QORJcH0n5UJXo0PararywOOh1xzgPdF7e6ymVfKYoyHpDj96YNTY
15:G+glEae6W/1XjA7vRm21nNyEco/c+J2TdR0Qp8dcjPLAhy3ui8kLEVaROwiiI6OeWu8qmX2iNj9yo/rTMtFzb6dz8xhQlKoBQI8fl9A304VnjFdz7MKPhw5PTrxsgHCk
... <skipped>
29:G+glEae6W/1XjA7vRm21nNyEco/c+J2TdR0Qp8dcjPLAhy3ui8kLEVaROwiiI6Oes5A4wo33m2XSYVHfWPfqox90cU12/MXUZMaiIebtmORiI6FNnEKRuYd1sD+8c9Tt2K5R19pxsrCD2Rmg17iLmA==
30:G+glEae6W/1XjA7vRm21nNyEco/c+J2TdR0Qp8dcjPLAhy3ui8kLEVaROwiiI6Oes5A4wo33m2XSYVHfWPfqo+zTb4/ZFk1ANUDkSXB9J+VCV6ND2q2q8sDjodcc4D3Re3usplXymKMh6Q4/emDU2A==
31:G+glEae6W/1XjA7vRm21nNyEco/c+J2TdR0Qp8dcjPLAhy3ui8kLEVaROwiiI6Oes5A4wo33m2XSYVHfWPfqo1rvKpl9ojY/cqP60zLRc2+nc/MYUJSqAUCPH5fQN9OFZ4xXc+zCj4cOT068bIBwpA==
... <skipped>
45:G+glEae6W/1XjA7vRm21nNyEco/c+J2TdR0Qp8dcjPLAhy3ui8kLEVaROwiiI6Oes5A4wo33m2XSYVHfWPfqo7OQOMKN95tl0mFR31j36qMfdHFNdvzF1GTGoiHm7ZjkYiOhTZxCkbmHdbA/vHPU7diuUdfacbKwg9kZoNe4i5g=
46:G+glEae6W/1XjA7vRm21nNyEco/c+J2TdR0Qp8dcjPLAhy3ui8kLEVaROwiiI6Oes5A4wo33m2XSYVHfWPfqo7OQOMKN95tl0mFR31j36qPs02+P2RZNQDVA5ElwfSflQlejQ9qtqvLA46HXHOA90Xt7rKZV8pijIekOP3pg1Ng=
47:G+glEae6W/1XjA7vRm21nNyEco/c+J2TdR0Qp8dcjPLAhy3ui8kLEVaROwiiI6Oes5A4wo33m2XSYVHfWPfqo7OQOMKN95tl0mFR31j36qNa7yqZfaI2P3Kj+tMy0XNvp3PzGFCUqgFAjx+X0DfThWeMV3Pswo+HDk9OvGyAcKQ=
... <skipped>
```

From the above output, we can have a few observations:

-    When looping through different letters, the first two blocks are always the same regardless of the input letter.
Similarly for the last two blocks. Only the block in the middle one keeps changing with the input letter.
That means our input is in between a constant prefix and suffix, and then encrypted.
-    When looping through different lengths, the number of the blocks in the output is increased once every additional 16 input characters (e.g., length = 13, 29, 45).
That means it uses a block cipher whose block size is 128 bits (e.g., AES).
-    The web application uses ECB (Electronic CodeBook) as its operation mode. Otherwise the change of previous block will impact the next block's output.
ECB is considered as a non-secure operation mode for block ciphers, especially for long repeated messages.


I was also given some other hints by my friend:
-    Very likely the query is done by a SQL query, e.g., `SELECT <column> FROM <table> WHERE <column> LIKE '%input%'`.
-    Very likely we can solve this challenge by SQL injection as long as we can figure out the starting/ending position of the constant prefix and suffix.
-    If we can figure out the starting/ending position, we then inject a SQL `UNION ALL SELECT password FROM users;#` between the prefix and suffix.
This injection will help us to show the password for all users in the table.


Thanks to Alexandre, I borrowed his solution and (kind of cheating) solved the problem.
It revealed that natas29's password is <small>airooCaiseiyee8he8xongien9euhe8b</small>.



<strong>CONCLUSION</strong>

Although I was able to tell the challenge is about cryptography,
I couldn't clear this level without the help of my friend and online resources.
There is always something new to learn.

To summarize this challenging, we learned a few things:
1. ECB is not a secure operation mode for block ciphers. Some patterns are remained even after an encryption, because each block is independent.
It is especially vulnerable for long repeated messages.
2. For sensitive messages, we should consider more secure operation mode, e.g., the counter mode.

<strong>References</strong>

1. <a href="https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation">Block cipher mode of operation</a>
2. <a href="https://vulncat.fortify.com/en/detail?id=desc.semantic.cpp.weak_encryption_insecure_mode_of_operation">Weak Encryption: Insecure Mode of Operation</a>


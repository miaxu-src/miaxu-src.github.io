---
category: natas
title: Natas9 - Command Injection Part 1
---

website: http://natas9.natas.labs.overthewire.org/ (password: W0mMhUcRRnG8dcghE4qvk3JA9lGt8nDl)

In this challenge, users can search a word in a dictionary by providing a keyword in a textbox. Here is the code snippet:

```php
<?
$key = "";

if(array_key_exists("needle", $_REQUEST)) {
    $key = $_REQUEST["needle"];
}

if($key != "") {
    passthru("grep -i $key dictionary.txt");
}
?>
```

As shown above, it uses the Linux `grep` command to achieve the goal. More important, the implementation uses the PHP function `passthru` to execute a Linux command.

In the PHP document, it mentions:
- <em>Warning: When allowing user-supplied data to be passed to this function, use escapeshellarg() or escapeshellcmd() to ensure that users cannot trick the system into executing arbitrary commands</em>.

The notice in the PHP document reminds me of command injection. Here we can come up with this input <small>; cat /etc/natas_webpass/natas10 #</small>. The output of the website given my input is:

```raw
    Output:
    nOpp1igQAkUzaI1GUUjzn1bFVj7xCNzu
```

As you can see, by injecting a Linux command `cat /etc/natas_webpass/natas10`, we successfully acquired the password for the next level.

Here I'm providing more details about the `passthru` and my input:

What the `passthru` does is that it uses the shell to launch a program and passes my input to execute to the shell. It leaves the task of breaking up the command’s arguments to the shell. As a result, the special characters in my input such as `;` and `#` are recognized and interpreted by the shell. In the Linux shell, the symbol `;` is a separator between two commands and the symbol `#` is a leading character of comments. As such, `passthru` considers there are two commands, one is the `grep` command and the other is the `cat /etc/natas_webpass/natas10` command.

<strong>CONCLUSION</strong>

Be careful with command injection, especially when you are dealing with user inputs. One should avoid using vulnerable functions such as `passthru` in this challenge. Also, all user inputs should be sanitized before they are sent for further processing.

<strong>References</strong>

- https://www.owasp.org/index.php/Command_Injection

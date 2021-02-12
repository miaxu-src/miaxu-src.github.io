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

The notice in the PHP document reminds me of command injection. In fact, the `grep` command can be used to grep multiple files, which are separated by spaces. With that information in mind, I provided the following input <small>. /etc/natas_webpass/natas10</small> in the textbox. The output of the website given my input is:

```raw
    Output:
    /etc/natas_webpass/natas10:nOpp1igQAkUzaI1GUUjzn1bFVj7xCNzu
    dictionary.txt:African
    dictionary.txt:Africans
    ... <skip lines below>
```

As you can see, the `grep` command has searched the wildcards <small>.</small> in the file <small>/etc/natas_webpass/natas10</small> as well due to the space between the wildcards and the file name.

<strong>CONCLUSION</strong>

Be careful with command injection, especially when you are dealing with user inputs. One should avoid using vulnerable functions such as `passthru` in this challenge. Also, all user inputs should be sanitized before they are sent for further processing.

<strong>References</strong>

- https://www.owasp.org/index.php/Command_Injection

---
category: natas
title: Natas10 - Command Injection Part 2
---

website: http://natas10.natas.labs.overthewire.org/ (password: nOpp1igQAkUzaI1GUUjzn1bFVj7xCNzu)

In this challenge, some special characters, e.g., `;`, the one we used in crack Natas9, are filtered to avoid command injection:

```php
if($key != "") {
    if(preg_match('/[;|&]/',$key)) {
        print "Input contains an illegal character!";
    } else {
        passthru("grep -i $key dictionary.txt");
    }
}
```

The above change makes our command injection not as easy as the previous one. However, if you are familiar with the usage of `grep`, you will quickly realize that this command can take multiple file names and thus look for a keyword in multiple files. With that information in mind, I provided the following input <small>. /etc/natas_webpass/natas11</small> in the textbox. The output of the website given my input is:

```raw
    Output:
    /etc/natas_webpass/natas11:U82q5TCMMQ9xuFoI3dYX61s7OZD9JKoK
    dictionary.txt:African
    dictionary.txt:Africans
    ... <skip lines below>
```

As you can see, the `grep` command has searched the wildcards <small>.</small> in two files, one is my input <small>/etc/natas_webpass/natas11</small> and the other is the dictionary file.

<strong>CONCLUSION</strong>

There are different ways to inject commands. Even though you filter out some special characters, you may still leave other security holes accessible. Therefore, avoid using vulnerable functions such as `passthru` is the ultimate direction we should go.

<strong>References</strong>

- https://www.owasp.org/index.php/Command_Injection

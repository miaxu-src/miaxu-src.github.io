---
category: natas
---
website: http://natas6.natas.labs.overthewire.org/ (password: aGoY4q2Dc6MgDq4oL4YtoKtyAg9PeHa1)

According to the PHP source code, it includes a secret file:
```php
<?
    include "includes/secret.inc";

    if(array_key_exists("submit", $_POST)) {
        if($secret == $_POST['secret']) {
            print "Access granted. The password for natas7 is <censored>";
        } else {
            print "Wrong secret";
        }
    }
?>
```

Pasting that file path into browser, you will get the secret. Submitting the secret, we have the password for natas7.

<strong>CONCLUSION</strong>

To summarize this level, I think there are a few vulnerabilities exposed:
1. <em>Sensitive data exposure</em>. The secret is saved in plaintext without any encryption or hash.
2. <em>Access control weakness</em>. The file that saves the secret can be accessed without any permission restriction.

Due to the above vulnerabilities, attackers can easily retrieve the secret info.

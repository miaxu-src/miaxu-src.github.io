---
category: natas
title: Natas8 - Info Disclosure and Reverse
---

website: http://natas8.natas.labs.overthewire.org/ (password: DBfUBfqQG69KvJvJ1iAbMoIpwSNQ9bWe)

A piece of PHP code is given in this level:

```php
<?
$encodedSecret = "3d3d516343746d4d6d6c315669563362";

function encodeSecret($secret) {
    return bin2hex(strrev(base64_encode($secret)));
}

if(array_key_exists("submit", $_POST)) {
    if(encodeSecret($_POST['secret']) == $encodedSecret) {
    print "Access granted. The password for natas9 is <censored>";
    } else {
    print "Wrong secret";
    }
}
?>
```

According to the source code above, we can learn two things:
1. The secret is saved encoded. Also, we know how it's encoded.
2. More important, the encoding process is <em>reversible</em>.

Therefore, it's possible for us to retrieve the secret. To retrieve it, we simply write a decode function by reversing the `encodeSecret`:

```php
<?
$encodedSecret = $argv[1];
echo base64_decode(strrev(hex2bin($encodedSecret)));
?>
```

Running this script gives us the secret <small>oubWYf2kBq</small>. Put the decoded secret in the textbox and I get the password for the next level.

<strong>CONCLUSION</strong>

In this challenge, we see that a secret is saved locally in an encoded way. Moreover, this secret can be reversed because the developer didn't use a right way to enclose it. Here, we can follow a few suggestions to enhance the security of secret information:
1. We should avoid hardcoding secret in our source code by any chance.
2. When storing a secret, we should consider a proper way, e.g., using a recognized strong hashing algorithm. This will help to prevent attackers from reversing the secret.

<strong>References</strong>

Here are some additional references about reversible one-way hash:
- <a href="https://cwe.mitre.org/data/definitions/328.html">CWE-328: Reversible One-Way Hash</a>
- <a href="https://cwe.mitre.org/data/definitions/759.html">CWE-759: Use of a One-Way Hash without a Salt</a>


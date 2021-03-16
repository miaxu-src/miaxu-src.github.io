---
category: natas
title: Natas11 - Broken Cryptography
---

website: http://natas11.natas.labs.overthewire.org (password: U82q5TCMMQ9xuFoI3dYX61s7OZD9JKoK)

Here is the PHP code of this challenge:
{% highlight php linenos %}
<?
$defaultdata = array( "showpassword"=>"no", "bgcolor"=>"#ffffff");
 
function xor_encrypt($in) {
    $key = '<censored>';
    $text = $in;
    $outText = '';
 
    // Iterate through each character
    for($i=0;$i<strlen($text);$i++) {
    $outText .= $text[$i] ^ $key[$i % strlen($key)];
    }
 
    return $outText;
}
 
function loadData($def) {
    global $_COOKIE;
    $mydata = $def;
    if(array_key_exists("data", $_COOKIE)) {
    $tempdata = json_decode(xor_encrypt(base64_decode($_COOKIE["data"])), true);
    if(is_array($tempdata) && array_key_exists("showpassword", $tempdata) && array_key_exists("bgcolor", $tempdata)) {
        if (preg_match('/^#(?:[a-f\d]{6})$/i', $tempdata['bgcolor'])) {
        $mydata['showpassword'] = $tempdata['showpassword'];
        $mydata['bgcolor'] = $tempdata['bgcolor'];
        }
    }
    }
    return $mydata;
}
 
function saveData($d) {
    setcookie("data", base64_encode(xor_encrypt(json_encode($d))));
}
 
$data = loadData($defaultdata);
 
if(array_key_exists("bgcolor",$_REQUEST)) {
    if (preg_match('/^#(?:[a-f\d]{6})$/i', $_REQUEST['bgcolor'])) {
        $data['bgcolor'] = $_REQUEST['bgcolor'];
    }
}
 
saveData($data);
?>
{% endhighlight %}

From the source code, we will have following observations:
1. A cookie is used to load a right page.
2. The cookie has a data field.
3. The data field is generated from something like `array("showpassword"=>"no", "bgcolor"=>"#ffffff")`.
4. Here is the way to generate the data field: `$data = base64_encode(xor_encrypt(json_encode($d))`.
5. If the "showpassword" in the array is "yes", then the password will show up on the page.

Now the question is how to send a request with "showpassword=yes"? To achieve this goal, we will need three steps:
1. Reverse the `$key` used in `xor_encrypt()` (i.e., line 5).
2. Using the `$key`, craft a cookie with `showpassword=yes`.
3. Send out a request using the crafted cookie.

For the 1st step, I first use Burp Suite to capture the pair of `$data` and `$d`:
- d: array("showpassword"=>"no", "bgcolor"=>"#ffffff")
- data: ClVLIh4ASCsCBE8lAxMacFMZV2hdVVotEhhUJQNVAmhSEV4sFxFeaAw=

I then used a similar php code to reverse the key:
```php
function xor_encrypt($in, $out) {
    $key = '';

    // Iterate through each character
    for ( $i = 0; $i < strlen($in); $i++ ) {
        $key .= $in[$i] ^ $out[$i % strlen($out)];
    }

    return $key;
}

// step 1: reverse the key
$d = array("showpassword"=>"no", "bgcolor"=>"#ffffff");
$plain = json_encode($d);
print "plain: $plain";

$data = "ClVLIh4ASCsCBE8lAxMacFMZV2hdVVotEhhUJQNVAmhSEV4sFxFeaAw=";
$cipher = base64_decode($data);
print "cipher: $cipher";

$key = xor_encrypt($plain, $cipher);
echo "key: $key\n\n";
```

For the 2nd step, I use the above method to craft a cookie from the object `array("showpassword"=>"yes", "bgcolor"=>"#ffffff")`. The generated cookie is "ClVLIh4ASCsCBE8lAxMacFMOXTlTWxooFhRXJh4FGnBTVF4sFxFeLFMK":
```php
// step 2: craft a cookie using the reversed key, which is "qw8J"
$key = "qw8J";
$craft = array("showpassword"=>"yes", "bgcolor"=>"#ffffff");
$cookie = base64_encode(xor_encrypt(json_encode($craft), $key));
echo "cookie: $cookie\n\n";
```

Lastly, I used a `curl` command to send out a request with my crafted cookie to retrieve the password:
```shell
$ curl -s http://natas11:U82q5TCMMQ9xuFoI3dYX61s7OZD9JKoK@natas11.natas.labs.overthewire.org -H 'cookie: data=ClVLIh4ASCsCBE8lAxMacFMOXTlTWxooFhRXJh4FGnBTVF4sFxFeLFMK'
... <skip>
Cookies are protected with XOR encryption<br/><br/>

The password for natas12 is EDXp0pS26wLKHZy1rDBPUZk0RKfLGIR3<br>
...<skip>
```

As you can see, we successfully retrieved the password for natas12.

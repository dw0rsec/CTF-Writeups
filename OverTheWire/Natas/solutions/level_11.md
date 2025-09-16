# level 11

**Host**: http://natas11.natas.labs.overthewire.org
>natas11:UJdqkK1pTu6VLt9UHWAgRZ**********

When you visit the site, you get the following message:

```
Cookies are protected with XOR encryption.
```

### Solution:

When you click on the "View source" link you get following php code:

```php
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

<?
if($data["showpassword"] == "yes") {
    print "The password for natas12 is <censored><br>";
}

?>
```

When you click on the `Set color` button eith the `ffffff` value you get following cookie.

```http
Set-Cookie: data=HmYkBwozJw4WNyAAFyB1VUcqOE1JZjUIBis7ABdmbU1GIjEJAyIxTRg%3D
```

So the url decoded value is:

```
HmYkBwozJw4WNyAAFyB1VUcqOE1JZjUIBis7ABdmbU1GIjEJAyIxTRg=
```

From the php code you know that you have to change the `showpassword=no` to `showpassword=yes` in the cookie.

```php
$ php -a
Interactive shell

php > echo base64_encode(json_encode(array( "showpassword" => "no", "bgcolor" => "#ffffff")));
```

This returns following value:

```
eyJzaG93cGFzc3dvcmQiOiJubyIsImJnY29sb3IiOiIjZmZmZmZmIn0=
```

You can use a tool like [CyberChef](https://gchq.github.io/CyberChef/) to XOR this value with the cookie value that was set by clicking on the `Set color` button. This will return the XOR key `eDWo`.

So now you can craft a cookie with following value:

```json
{"showpassword":"yes","bgcolor":"#ffffff"}
```

and then XOR it with our key and base64 encode it to get following cookie:

```
HmYkBwozJw4WNyAAFyB1VUc9MhxHaHUNAic4Awo2dVVHZzEJAyIxCUc5
```

Just send a request with our new cookie and grab the password for natas12 from the response.

```html
HTTP/1.1 200 OK
Date: Thu, 04 Sep 2025 10:06:00 GMT
Server: Apache/2.4.58 (Ubuntu)
Set-Cookie: data=HmYkBwozJw4WNyAAFyB1VUc9MhxHaHUNAic4Awo2dVVHZzEJAyIxCUc5
Vary: Accept-Encoding
Content-Length: 1149
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive
Content-Type: text/html; charset=UTF-8

<html>
<head>
<!-- This stuff in the header has nothing to do with the level -->
</head>

<h1>natas11</h1>
<div id="content">
<body style="background: #ffffff;">
Cookies are protected with XOR encryption<br/><br/>

The password for natas12 is yZdkjAYZRd3R7tq7T5kXMj**********<br>
<form>
Background color: <input name=bgcolor value="#ffffff">
<input type=submit value="Set color">
</form>

<div id="viewsource"><a href="index-source.html">View sourcecode</a></div>
</div>
</body>
</html>
```
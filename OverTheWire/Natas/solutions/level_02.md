# level 2

**Host**: http://natas2.natas.labs.overthewire.org
>natas2:TguMNxKo1DSa1tujBLuZJnDUlCcUAPlI

When you visit the site, you get the following message:

```
There is nothing on this page.
```

### Solution:

You can find the `/files` directory in the source code where the `pixel.png` is located.

```html
HTTP/1.1 200 OK
Date: Wed, 03 Sep 2025 16:02:40 GMT
Server: Apache/2.4.58 (Ubuntu)
Last-Modified: Fri, 15 Aug 2025 13:06:35 GMT
ETag: "368-63c670f45ed3d-gzip"
Accept-Ranges: bytes
Vary: Accept-Encoding
Content-Length: 872
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive
Content-Type: text/html

<html>
<head>
<!-- This stuff in the header has nothing to do with the level -->
</head>
<body>
<h1>natas2</h1>
<div id="content">
There is nothing on this page
<img src="files/pixel.png">
</div>
</body></html>
```

When you inspect the `/files` directory you can find a `users.txt` containing the password for natas3.

```
# username:password
alice:BYNdCesZqW
bob:jw2ueICLvT
charlie:G5vCxkVV3m
natas3:3gqisGdR0pjm6tpkDKdIWO2hSvchLeYH
eve:zo4mJWyNj2
mallory:9urtcpzBmH
```
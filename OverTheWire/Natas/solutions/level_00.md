# level 0		

**Host**: http://natas0.natas.labs.overthewire.org
>natas0:natas0

When you visit the site, you get the following message:

```
You can find the password for the next level on this page.
```

### Solution:

View the source code in your browser or grab the password in the response in burpsuite.

```html
HTTP/1.1 200 OK
Date: Wed, 03 Sep 2025 15:43:52 GMT
Server: Apache/2.4.58 (Ubuntu)
Last-Modified: Fri, 15 Aug 2025 13:06:35 GMT
ETag: "396-63c670f3f9268-gzip"
Accept-Ranges: bytes
Vary: Accept-Encoding
Content-Length: 918
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive
Content-Type: text/html

<html>
<head>
<!-- This stuff in the header has nothing to do with the level -->
</head>
<body>
<h1>natas0</h1>
<div id="content">
You can find the password for the next level on this page.

<!--The password for natas1 is 0nzCigAq7t2iALyvU9xcHl********** -->
</div>
</body>
</html>
```
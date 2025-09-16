# level 1

**Host**: http://natas1.natas.labs.overthewire.org
>natas1:0nzCigAq7t2iALyvU9xcHl**********

When you visit the site, you get the following message:

```
You can find the password for the next level on this page, but rightclicking has been blocked! 
```

### Solution:

Just press `Cmd+U` (on macos) or just grab the password from the response in your burpsuite.

```html
HTTP/1.1 200 OK
Date: Wed, 03 Sep 2025 15:58:10 GMT
Server: Apache/2.4.58 (Ubuntu)
Last-Modified: Fri, 15 Aug 2025 13:06:35 GMT
ETag: "427-63c670f40d5c5-gzip"
Accept-Ranges: bytes
Vary: Accept-Encoding
Content-Length: 1063
Keep-Alive: timeout=5, max=99
Connection: Keep-Alive
Content-Type: text/html

<html>
<head>
<!-- This stuff in the header has nothing to do with the level -->
</head>
<body oncontextmenu="javascript:alert('right clicking has been blocked!');return false;">
<h1>natas1</h1>
<div id="content">
You can find the password for the
next level on this page, but rightclicking has been blocked!

<!--The password for natas2 is TguMNxKo1DSa1tujBLuZJn********** -->
</div>
</body>
</html>
```
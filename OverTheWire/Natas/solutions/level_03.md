# level 3

**Host**: http://natas3.natas.labs.overthewire.org
>natas3:3gqisGdR0pjm6tpkDKdIWO**********

When you visit the site, you get the following message:

```
There is nothing on this page.
```

### Solution:

By inspecting the source you find a hint that 'not even google will find it', which suggests that one should look at `robots.txt`.

```html
HTTP/1.1 200 OK
Date: Wed, 03 Sep 2025 16:23:51 GMT
Server: Apache/2.4.58 (Ubuntu)
Last-Modified: Fri, 15 Aug 2025 13:06:35 GMT
ETag: "39b-63c670f43ce4c-gzip"
Accept-Ranges: bytes
Vary: Accept-Encoding
Content-Length: 923
Keep-Alive: timeout=5, max=99
Connection: Keep-Alive
Content-Type: text/html

<html>
<head>
<!-- This stuff in the header has nothing to do with the level -->
</head>
<body>
<h1>natas3</h1>
<div id="content">
There is nothing on this page
<!-- No more information leaks!! Not even Google will find it this time... -->
</div>
</body></html>
```

The `robots.txt` reveals the `/s3cr3t` direcory, which contains the `user.txt` with the password for natas4.

```shell
# robots.txt
User-agent: *
Disallow: /s3cr3t/
```

```shell
# users.txt
natas4:QryZXc2e0zahULdHrtHxzy**********
```
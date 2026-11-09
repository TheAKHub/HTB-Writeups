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
The `app.py` source code does prove the XML and XSLT vulnerability attempted previously.
###### Vulnerable Code
```
xslt_file = request.files['xslt_file'] #accepts the XSLT from the user

xslt_tree = etree.parse(xslt_path)

transform = etree.XSLT(xslt_tree)
result_tree = transform(xml_tree)
```

. the install.dm, however, shows that there is a cronjob that runs any python script located in the scripts folder every 60 seconds.

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

We see that underneath the area where the IP is located, we see a changelog 
<img width="677" height="520" alt="image" src="https://github.com/user-attachments/assets/c457dedf-bc4e-43ed-a989-c34b035e72fa" />

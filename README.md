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

* We see that the server is leaking inodes via ETags in the header of /robots.txt. This relates to the CVE-2003-1418 vulnerability. These Entity Tags are an HTTP header which are used for Web cache validation and conditional requests from browsers for resources.
* Apache mod_negotiation is enabled with MultiViews, which will allow us to use a brute force attack in order to discover existing files on a server which uses mod_negotiation.
* The following alternatives for ‘index’ were found: index.html, and index.php. These can be used to provide us with more info on the website.
* OSVDB-3092: /admin/: This might be interesting… if we have a login. Good to keep that in the back of our mind.
- /admin/index.html: Admin login page/section found - also relates to the above scan.
* /readme.html: This WordPress file reveals the installed version.
- Basically tells us that this is a WordPress Site! So we know we can look for WordPress Vulnerabilities.
- /wp-links-opml.php: This WordPress script reveals the installed version.
- /wp-login/: Admin login page/section found.
- /wp-admin/wp-login.php: Wordpress login found.
* OSVDB-3092: /license.txt: License file found may identify site software. Which can help us get version information of plugins and services to look for exploits.

Alright, we got our initial footprint, let’s go ahead and access the website in our browser by navigating to 192.168.1.9.
![screenshot](images/4.jpg)

es - I came here for a reason, to hack you! Anyways, that website is actually pretty freakin cool!
We can see that we are able to run 6 commands in the interface, each does its own little thing. So go ahead and play around with them - I did, and thoroughly enjoyed it - but, let’s get back to the CTF!
We already know that there are leaking indoes via ETags at /robots.txt, which is basically a text file that is used to prevent crawlers from indexing portions of the website. Let’s go ahead and navigate to http://192.168.1.9/robots.txt.

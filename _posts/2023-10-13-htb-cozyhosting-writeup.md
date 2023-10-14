---
title: HTB Cozyhosting Writeup
date: 2023-10-13 09:20:16 +0600
categories: [Linux, Easy]
tags: [htb]     
---

![Cozyhosting Pwned](https://raw.githubusercontent.com/ImdadMiran17/ImdadMiran17.github.io/main/assets/img/cozyhosting-htb/cozyhosting_pwned.PNG "Pwned haha!!")

# Introduction
Hi, Hackers!!. Today we will solve the HTB machine Cozyhosting. It is a fun linux machine. And also tagged as Easy. Let's dive into the fun stuff
without wasting any time.

# Enumeration
Just like as usual, when you approach a machine or box, first thing to do is to scan for open ports. For scanning ports, you can use the traditional tool `Nmap`. But `Nmap` is kinda slow. As an alternative, we can use `Rustscan` or `threader3000`. I used `Rustscan` for scanning. Found the ports 22 and 80 open.

![Cozyhosting Rustscan](https://raw.githubusercontent.com/ImdadMiran17/ImdadMiran17.github.io/main/assets/img/cozyhosting-htb/cozyhosting_rustscan.png)

And also found what services are running in those open ports.

![Cozyhosting Services](https://raw.githubusercontent.com/ImdadMiran17/ImdadMiran17.github.io/main/assets/img/cozyhosting-htb/cozyhosting_nmap_service.png)

If you search with the given IP in your browser, you will be redirected to `http://cozyhosting.htb`. So we need to add the IP and hostname in the hosts file. For linux, it's `/etc/hosts` and for windows, it's `c:\Windows\System32\Drivers\etc\hosts`.

Then we get,

![Home Page](https://raw.githubusercontent.com/ImdadMiran17/ImdadMiran17.github.io/main/assets/img/cozyhosting-htb/cozyhosting_port80.png)

And We found a login page too.

![Login Page](https://raw.githubusercontent.com/ImdadMiran17/ImdadMiran17.github.io/main/assets/img/cozyhosting-htb/cozyhosting_login.png)
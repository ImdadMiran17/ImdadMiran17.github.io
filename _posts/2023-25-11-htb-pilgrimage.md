---
title: HTB Pilgrimage Writeup
date: 2023-11-25 13:15:16 +0600
categories: [Linux, Easy]
tags: [htb] 
toc: true    
---

![Pilgrimage Pwned](https://raw.githubusercontent.com/ImdadMiran17/ImdadMiran17.github.io/main/assets/img/pilgrimage-htb/pwned.png "Pwned haha!!")

# Introduction
Bonjour, passionné de cybersécurité :) There is a linux machine named **Pilgrimage** in Hack The Box. Let's give it a try, shall we?

# Enumeration
Scan the IP with nmap.

```bash
sudo nmap -sS -T4 -n -v -A 10.10.11.219 > nmap_discovered_ports
```

```bash
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.4p1 Debian 5+deb11u1 (protocol 2.0)
| ssh-hostkey: 
|   3072 20:be:60:d2:95:f6:28:c1:b7:e9:e8:17:06:f1:68:f3 (RSA)
|   256 0e:b6:a6:a8:c9:9b:41:73:74:6e:70:18:0d:5f:e0:af (ECDSA)
|_  256 d1:4e:29:3c:70:86:69:b4:d7:2c:c8:0b:48:6e:98:04 (ED25519)
80/tcp open  http    nginx 1.18.0
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Did not follow redirect to http://pilgrimage.htb/
|_http-server-header: nginx/1.18.0
```

The IP redirects to `http://pilgrimage.htb/`. So we need to add the IP and hostname in the hosts file. For linux, it's `/etc/hosts` and for windows, it's `c:\Windows\System32\Drivers\etc\hosts`.

Looks like it's a image shinker.

![Home Page](https://raw.githubusercontent.com/ImdadMiran17/ImdadMiran17.github.io/main/assets/img/pilgrimage-htb/home_page.PNG)

It shrinks images and give a link to access the image. If only we could know the software shrinking the images...
Now, run `dirsearch` to directory traversing. `.git` directory was found among the results. We have a git repository to dump. I used [GitDump](https://github.com/Ebryx/GitDump) to dump the repository.

![Git Dump](https://raw.githubusercontent.com/ImdadMiran17/ImdadMiran17.github.io/main/assets/img/pilgrimage-htb/git_dump.png)

Guess what! we found the software that's shrinking the images underneath. It's `Imagemagick 7.1.0-49`. 

![Magick](https://raw.githubusercontent.com/ImdadMiran17/ImdadMiran17.github.io/main/assets/img/pilgrimage-htb/magick.png)

Also, we found the source codes. The sqlite database is in `/var/db/pilgrimage`.

![login.php](https://raw.githubusercontent.com/ImdadMiran17/ImdadMiran17.github.io/main/assets/img/pilgrimage-htb/database_recon.png)
![Commit Msg](https://raw.githubusercontent.com/ImdadMiran17/ImdadMiran17.github.io/main/assets/img/pilgrimage-htb/from_git_commit_msg.png)

# Foothold
Google about the version of Imagemagick for poc. You may find this [one](https://github.com/Sybil-Scan/imagemagick-lfi-poc). The python script of this poc will generate a exploit png with the payload which is a local directory. First create an image with `/etc/passwd`. Upload the image on the website. Download the modified image and procede with the command `identify -verbose result.png`.

```bash
-emily:x:1000:1000:emily,,,:/home/emily:/bin/bash
systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin
sshd:x:105:65534::/run/sshd:/usr/sbin/nologin
_laurel:x:998:998::/var/log/laurel:/bin/false
```

Now, create an image with `/var/db/pilgrimage`. After uploading it on the web, you will get the db file and the password for `emily`. I used [CyberChef](https://cyberchef.org/).

![Found Pass](https://raw.githubusercontent.com/ImdadMiran17/ImdadMiran17.github.io/main/assets/img/pilgrimage-htb/found_the_pass_sqlite.png)

You can log into ssh with `emily`. You know where is the user flag.

![SSH](https://raw.githubusercontent.com/ImdadMiran17/ImdadMiran17.github.io/main/assets/img/pilgrimage-htb/got_ssh.png)

# Priviledge Escalation
Now, you can run basic linux commands to find a way to priviledge escalation or upload `linpeas` on the machine. I uploaded `linpeas` and found this process running which looks suspicious. 

![Suspicious](https://raw.githubusercontent.com/ImdadMiran17/ImdadMiran17.github.io/main/assets/img/pilgrimage-htb/processes.png)

Let's see the source code of `/usr/sbin/malwarescan.sh`.

![Malwarescript](https://raw.githubusercontent.com/ImdadMiran17/ImdadMiran17.github.io/main/assets/img/pilgrimage-htb/malwarescan_sh.png)

The script will extract image files of `/var/www/pilgrimage.htb/shrunk/` with `binwalk` and look for blacklisted scripts in the files. The version of `binwalk` that's used in the machine is 2.3.2. Let's do some googling with that. You might find this [poc](https://www.exploit-db.com/exploits/51249) from `exploitdb`.

![Binwalk Version](https://raw.githubusercontent.com/ImdadMiran17/ImdadMiran17.github.io/main/assets/img/pilgrimage-htb/binwalk.png)

Create the exploit image with the your `machine ip` and `port` that you are listening on your machine. Put the image in `/var/www/pilgrimage.htb/shrunk/` and wait for `binwalk` to access it. You'll get the shell on your machine as root. You know where is the root flag.

![Rooted](https://raw.githubusercontent.com/ImdadMiran17/ImdadMiran17.github.io/main/assets/img/pilgrimage-htb/rooted.png)

##### Happy Hacking:)
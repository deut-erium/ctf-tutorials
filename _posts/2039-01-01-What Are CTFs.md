---
tags: introduction ctf
aside:
  toc: true
sidebar:
  nav: layouts
author: deuterium
excerpt_separator: <!--more-->
key: whatarectfs000001
---

CTF or [Capture The Flag](https://en.wikipedia.org/wiki/Capture_the_flag#Computer_security) in context of computer security, are special kind of information security competitions which serve as a gamified educational exercise to give participants experience in securing a machine as well as conducting and reacting to sort of attacks found in real world or some (supposedly) fun programming concept otherwise.
<!--more-->

## What are Flags which we are capturing?
Flags are often secret strings hidden in purposefully-vulnerable programs, websites or challenges. The challengers have to steal/hack the flag from the organizers of CTF event (as in Jepoardy format) or from other challengers (attack/defence format). The flags serve as a currency or proof of work for solving a given challenge which one can exchange for points :wink:  
A typical flag may look like `flag{s0m3_puN_0r_c0mm3n7_r3l473d_70_7h3_ch4ll3n63}`  

<div>{%- include extensions/youtube.html id='8ev9ZX9J45A' -%}</div>
  
<div>{%- include extensions/youtube.html id='Lus7aNf2xDg' -%}</div>

## Types of CTFs
As crazy as hacking can get, so can get the competitions. CTFs are based on various formats. Each format has its own timeline, skillset, category and difficulty of interest.

### Jeopardy
Yes somewhat similar to [the show](https://en.wikipedia.org/wiki/Jeopardy!). These can be considered similar to programming contests as all the competiting teams do not attack each other, but rather solve challenges.  
Each challenge has its own weightage in terms of points. Once a team solve a challenge, they are awarded the assigned number of points for the challenge.  
The points for a challenge are either pre-defined by the organizers based on difficulty or can be dynamically decreasing with the number of solves.  
These kind of CTFs are the most common. One can typically find atleast two jeopardy style ctfs each [weekend](https://ctftime.org/event/list/upcoming)  
[picoCTF](https://picoctf.com/) is one notable example and servers are always (hopefully) up.
A jeopardy ctf event typically spans two days but there are events which may span weeks or maybe months.  

### Attack Defence CTFs
In an attack/defense style competitions, each team is given a machine (or a small network) to defend, typically on an isolated competition network.  
Teams are scored on both their success in defending their assigned machine(s) and on their success in attacking the other team's machines. A variation from classic flag-stealing is to "plant" own flags on opponent's machines.  
These kinds of events are less frequent and typically run only for a few hours.  

<div>{%- include extensions/youtube.html id='RXgp4cDbiq4' -%}</div>

### King Of The Hill
King of the Hill (KoTH), as the name suggests, is a competitive hacking game, where teams compete against other teams to compromise a machine and then patch its vulnerabilities to stop other players from also gaining access.  
The longer a team maintains access, the more points it gets.  
These are typically very short (typically an hour) and have strong emphasis on penetration testing. The most interesting aspect of these CTFs is that they can be livestreamed and spectated. Whats more fun than seeing hacking live!   
[TryHackMe](https://tryhackme.com/games/koth) has been promoting and organizing these events and one can join a game every 15 minutes!  

## Categories
CTFs cover a wide spectrum of aspects related to cybersecurity and programming in general. Note that sometimes a clear distinction between category can not be made as tasks require a mixed set of skills.

### Cryptography
[Cryptography](https://en.wikipedia.org/wiki/Cryptography) is a study of secrets. These CTF challenges can cover anything from some old classical cipher(aka caesar) and encodings, breaking self-rolled/poorly designed or implemented cryptographic protocols to implementing new cryptographic attacks based on recent papers/publications. These challenges can involve heavy mathematical and theoretical concepts. 

### Reverse Engineering
[Reverse Engineering](https://en.wikipedia.org/wiki/Reverse_engineering) is the process by which a man-made object is deconstructed to reveal its designs, architecture, code or to extract knowledge from the object; similar to scientific research, the only difference being that scientific research is about a natural phenomenon.  
What can be engineered can definitely be reverse-engineered and hence re-engineered too! These set of challenges often involve breaking authentication mechanisms or simply finding an input which gives a desried outcome. Be it lisence cracking, malware analysis, game hacking everything is a subset of reverse-engineering.

### Binary Exploitation
Binary Exploitation is a broad topic within Cyber Security which really comes down to finding a vulnerability in the program and exploiting it to gain control of a shell or modifying the program's functions. The binaries or executables involved are typically ELF or windows binary running on some server.

### Web Exploitation
Web exploitation challenges deal with security of web application often programmed in a variety of languages. The user is expected to find and leverage a vulnerability in the website to gain a higher privilege level.

### OSINT (Open Source INTelligence)
[Open Source Intelligence](https://en.wikipedia.org/wiki/Open-source_intelligence) is the collection and analysis of information that is gathered from public, or open, sources. These usually involve the discovery of some information about a target from their online presence and history.  

### Forensics
Forensics is the art of recovering the digital trail left on a computer. There are plently of methods to find data which is seemingly deleted, not stored, or worse, covertly recorded.  
These set of challenges usually involve smart enumeration to reveal patterns of required information. Be it picking out data of network dumps, or extracting information from corrupted files, forensics requires a lot of creativity and out-of-box thinking.

### Steganography
[Steganography](https://en.wikipedia.org/wiki/Steganography) is the practice of concealing a file, message, image, or video within another file, message, image, or video. To be concise, hiding information from seemingly plain sight. Challenges usually involve discoverying a file, message or information from provided files. 

### Penetration Testing
Penetration test is an authorized simulated cyberattack on a computer system, performed to evaluate the security of the system. Challenges usually involve a remote machine to explore and exploit.  

### Mobile Security
These set of challenges are involved with the security aspects of ever growing mobile technologies. Be it simply reverse engineering an android application or modifying and resigning it with malicious patch of code, this is more like reverse engineering for mobile based applications.

### Game Hacking
These set of problems deal with the techniques ususally involved in hacking a game. There would usually be a game on completing which, a flag can be obtained.  
Rest is up to participants' creativity! Be it simply searching through game files, modifying game binary or memory addresses to one's advantage or fooling the game server, the possibilities are limitless and limitlessly fun.

### Programming?
CTFs directly or indirectly involve a lot of programming, scripting and automation tasks. Be it repeatedly requesting a website, interacting with a socket server, some cool mathematical game or puzzle or any other generic task which can take years by hand!  

For more information check out this detailed video  

<div>{%- include extensions/youtube.html id='9WhQUItNNMw' -%}</div>


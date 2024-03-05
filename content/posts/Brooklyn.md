---
title: "Brooklyn Nine-Nine"
date: 2023-11-2T09:13:36-06:00
draft: false
toc: false
images:
tags:
  - tryhackme
---

### Nmap

![[Pasted image 20231006021416.png]]

`Nmap scan service version and script as default`

![[Pasted image 20231006021607.png]]

We have 3 Ports 
1. 21 :- FTP
2. 22 :- SSH
3. 80 :- HTTP

## Port 21 

###### FTP login using Anonymous 

![[Pasted image 20231006023952.png]]

![[Pasted image 20231006024047.png]]

`we know that the username is `Amy`and` Jake`we will try to use brute force and break it.

**IN PORT 21, WE GET USERNAMES**
1. AMY
2. JAKE
3. HOLT

this can be use for SSH brute-forcing using HYDRA

## Port 80
Here we found an image

we have an image this one : - 
![[brooklyn99.jpg]]

we found noting in metadata of this image :- 
![[Pasted image 20231006032921.png]]

when we try to find data hidden inside the image then it asked for password :- 
![[Pasted image 20231006033007.png]]

we will use `stegcracracker` to get the pass word:- 
![[Pasted image 20231006033104.png]]

we get the password as `admin`
![[Pasted image 20231006033559.png]]

now we will use `steghide` to crack the data hidden in it.
![[Pasted image 20231006033611.png]]

The password that is hidden is under note.txt 
![[Pasted image 20231006033837.png]]

## Port 22
#### Using Hydra For SSH
![[Pasted image 20231006035002.png]]
#### Login To SSH

![[Pasted image 20231006035427.png]]

we get inside the jake ssh but we have nothing so we will use the other users
![[Pasted image 20231006035511.png]]

we have 3 users (we already know this in FTP, note.txt )

we have holt password which we have cracked using `stegcrack` and `steghide`
so we will use `Sudo su holt`

we get our first flag
![[Pasted image 20231006035632.png]]

for privEsc we can use `sudo -l`
![[Pasted image 20231006040435.png]]
![[Pasted image 20231006041113.png]]

![[Pasted image 20231006040451.png]]
![[Pasted image 20231006040510.png]]

or we will get ![[Pasted image 20231006041206.png]]

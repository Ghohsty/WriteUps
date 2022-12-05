![[Pasted image 20220222192252.png]]
#sqlinjection

NMAP Scan
```
Starting Nmap 7.92 ( https://nmap.org ) at 2022-02-22 18:59 MST
Nmap scan report for 10.129.50.78
Host is up (0.066s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-title: Login
|_http-server-header: Apache/2.4.38 (Debian)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.97 seconds
```

Default login check on the main page (admin/admin, admin/root, etc)

GoBuster Scan
Nothing useful here
```
┌──(ghohst㉿kali)-[~]
└─$ sudo gobuster dir -e -u http://10.129.50.78/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt                
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.129.50.78/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Expanded:                true
[+] Timeout:                 10s
===============================================================
2022/02/22 19:07:13 Starting gobuster in directory enumeration mode
===============================================================
http://10.129.50.78/images               (Status: 301) [Size: 313] [--> http://10.129.50.78/images/]
http://10.129.50.78/css                  (Status: 301) [Size: 310] [--> http://10.129.50.78/css/]   
http://10.129.50.78/js                   (Status: 301) [Size: 309] [--> http://10.129.50.78/js/]    
http://10.129.50.78/vendor               (Status: 301) [Size: 313] [--> http://10.129.50.78/vendor/]
http://10.129.50.78/fonts                (Status: 301) [Size: 312] [--> http://10.129.50.78/fonts/] 
                                                                                                    
===============================================================
2022/02/22 19:16:19 Finished
===============================================================
```

- The site itself is vulnerable to a form of SQL injection whereby entering in '# and commenting out the remainder allows us to enter in any password. This is simply by luck, there happens to be an admin account and we're commenting out anything after (the password) therefore confusing the authentication and allowing us to enter
![[Pasted image 20220222192215.png]]

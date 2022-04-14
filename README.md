# 42Hive_Roger-Skyline
## 🎯Target

This subject aims to initiate you to the basics about system and network administration,  install a Virtual Machine and learn how to configure a web server as well as a lots of services used on a server machine.

🤨Rule on this project:
• You can choose which Linux OS you want to use in this subject. No use of techs like Traefik, no containers like Docker/Vagrant/etc...

## V1. VM Part

You have to run a Virtual Machine (VM) with the Linux OS of your choice (Debian Jessie, CentOS 7...) in the hypervisor of your choice (VMWare Fusion, VirtualBox...).

• A disk size of 8 GB.
• Have at least one 4.2 GB partition.
• It will also have to be up to date as well as the whole packages installed to meet the demands of this subject.

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
sudo -l -U test
  User test may run the following commands on this host:
     (ALL : ALL) ALL
```

If the user don't have sudo access, it will print that a user is **not allowed** to run `sudo` on localhost:

```bash
 sudo -l -U test
   User test is not allowed to run sudo on localhost.
```

### ****We don't want you to use the DHCP service of your machine. You've got to configure it to have a static IP and a Netmask in \30.****

2 adapters

1. NAT
2. Host-only Adapter

### You have to change the default port of the SSH service to one of your choices. SSH access HAS TO be done with publickeys. SSH root access SHOULD NOT be allowed directly, but with a user who can be root.

 Uncomment line `# Port 22` and change to a port between `49152 - 65535`. I chose 50000, easy to remember.

**[List of TCP and UDP port numbers](https://en.wikipedia.org/wiki/List_of_TCP_and_UDP_port_numbers)**

⭐**Dynamic ports** —Ports in the range 49152 to 65535 are not assigned, controlled, or registered. They are used for temporary or private ports. They are also known as private or non-reserved ports.

```bash
# Change the config file
sudo nano /etc/ssh/sshd_config

# Save and restart
sudo service sshd restart
```

Now can login via ssh on VM terminal

```bash
# It will prompt to create an ECDSA key and chose yes
sudo ssh test@10.0.2.15 -p 50000
```

Make sure the connection is active

```bash
sudo systemctl status ssh
# or
sudo service ssh status
```

If you work on school computer then you might already have RSA key so don't need to re-create, however if you do not then:

```bash
# host/computer terminal
ssh-keygen -t rsa
```

Now on your own computer log in to the VM:

```
$ ssh test@192.168.56.2 -p 50000
```

When you are done, logout by 

`exit`

The next step is important to secure our system. Otherwise this could be a hole where hackers could get in easily and modify/delete services, take important datas away

[Disable ssh login for root user](https://mediatemple.net/community/products/dv/204643810/how-do-i-disable-ssh-login-for-the-root-user)

```bash
sudo nano /etc/ssh/sshd_config
# change 
# PermitRootLogin Yes
# to
PermitRootLogin No
```

### You have to set the rules of your firewall on your server only with the services used outside the VM.

For this we use `ufw` (Uncomplicated FireWall)

Installing and activating the firewall:

`$ sudo apt install ufw
$ sudo ufw status
$ sudo ufw enable`

now that we installed our [Firewall](https://opensource.com/article/18/9/linux-iptables-firewalld) we need to [allow](https://help.ubuntu.com/community/UFW) for our port to be able to go through, also for later the optional task we gonna need http services.

`$ sudo ufw allow 55556/tcp
$ sudo ufw allow 80/tcp
$ sudo ufw allow 442/tcp`

now if we check the firewall status we can see these ports are allowed

`$ sudo ufw status`

**Networking protocols.**

FTP(file transfer protocol) requires port `20 or 21`

ssh requires port `22`

http requires port `80`

https requires port `443`

### You have to set a DOS (Denial Of Service Attack) protection on your open ports of your VM.

For different type of attack has been created different kind of security services We gonna use

[Fail2ban](https://en.wikipedia.org/wiki/Fail2ban), to secure our server

[iptables](https://en.wikipedia.org/wiki/Iptables), to configure and filter out dangerous IPs

[apache2](https://www.hostinger.com/tutorials/what-is-apache), to deploy our website in the comming task

`$ sudo apt install iptables fail2ban apache2`

After the installation we make a copy from jail.conf file to jail.local

The reason is that, beacuse with upgrades it might be modified so we want to keep the present one to keep our changes.

`$ sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
$ sudo vim /etc/fail2ban/fail2ban.local`

Here we scroll down or write to the search bar (:/sshd) to locate `# SSH servers`.

adding more rules:

`enable = true
maxentry = 3
bantime = 600`

[Check out for more details](https://www.tothenew.com/blog/fail2ban-port-80-to-protect-sites-from-dos-attacks/)

Now we write rules to protect port 80 (http) (our future website from dos attacks) search for `HTTP`

add:

`# protect port 80 (HTTP)

[http-get-dos]

enabled = true
port = http, https
filter = http-get-dos
logpath = %(apache_error_log)s
maxentry = 300
findtime = 300
bantime = 600
action = iptables[name=HTTP, port=http, protocol=tcp]`

Go to: `/ect/fail2ban/filter.d/http-get-dos.conf`

This gonna be our filter, which will filter out the attacks, so via this keep the server ports secured.

Add:

`# Fail2Ban configuration file

[Definition]

failregex = ^<HOST> -.*"(GET|POST).*
ignoreregex =`

[For more detail check out this](https://serverfault.com/questions/725936/fail2ban-regex-filter-doesnt-work-with-nginx-log-files)[, this](https://docs.python.org/2/library/re.html), [and this of course.](https://gist.github.com/SamStudio8/92507ad3e317edb9b869c20bb2623fcf)

after you made the changes we need to restart the firewall:

`$ sudo ufw reload
$ sudo service fail2ban restart`

now you make sure everything is setted up so:

`$ sudo fail2ban-client status`

### You have to set a protection against scans on your VM’s open ports.
[](https://en.wikipedia.org/wiki/List_of_TCP_and_UDP_port_numbers)

I installed a ports scan security service `PSAD`.

[here is a small guide how to set up basic stuff on it](https://www.unixmen.com/how-to-block-port-scan-attacks-with-psad-on-ubuntu-debian/)

However in the configuration file, there is a very good explanation for everything, and I made changes accordingly which I am not explaining here now.

### Stop the services you don’t need for this project.

all the services [you will find](https://linuxhint.com/disable_unnecessary_services_debian_linux/) in `/etc/init.d`

`$ ls /etc/init.d`

example I don't need console setup, because that's just responsible for fonts size etc.. so with disable i m gonna delete it:

`$ sudo systemctl disable console-setup.service`

### Create a script that updates all the sources of package, then your packages and which logs the whole in a file named /var/log/update_script.log. Create a scheduled task for this script once a week at 4AM and every time the machine reboots.

Create a script: `nano update.sh`

`#!/bin/bash
sudo apt-get update -y >> /var/log/update.log
sudo apt-get upgrade -y >> /var/log/update.log`

Give it permissions: `sudo chmod 755 update.sh` 755 means read and execute access for everyone and also write access for the owner of the file.

To automate execution, we must edit the crontab file: `sudo crontab -e` To which we add the lines:

```
0 4 * * 0 sudo ~scripts/update.sh
@reboot sudo ~/scripts/update.sh
```

This execute script every Sunday 4AM and when reboot

[Useful link](https://crontab.guru/#0_4_*_*_MON), and also [this](https://phoenixnap.com/kb/crontab-reboot).

### Make a script to monitor changes of the /etc/crontab file and sends an email to root if it has been modified. Create a scheduled script task every day at midnight.

[https://www.cyberciti.biz/faq/delete-all-root-email-mailbox/](https://www.cyberciti.biz/faq/delete-all-root-email-mailbox/) Install mail: `sudo apt install mailutils`

`sudo touch /etc/cron_monitor.sh`

then:

```bash
#!/bin/bash
DIFF=$(diff /etc/crontab.back /etc/crontab)
cat /etc/crontab > /etc/crontab.back
if [ "$DIFF" != "" ]; then
        echo "crontab check: changed, inform admin." | mail -s "crontab modified" root
fi
```

Give it permissions: `sudo chmod 755 cron_monitor.sh`

Edit crontab by adding the following line:

`0 0 * * * sudo ~/scripts/cron_monitor.sh`

To read simply type the following command: `mail` OR `mailx`

[Useful link](https://www.cmsimike.com/blog/2011/10/30/setting-up-local-mail-delivery-on-ubuntu-with-postfix-and-mutt/)

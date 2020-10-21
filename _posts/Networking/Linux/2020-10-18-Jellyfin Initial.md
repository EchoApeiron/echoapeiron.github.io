---
layout: post
title: Jellyfin, Setting up the Server
tags: linux networking jellyfin basic-security
---

Recently I learned about a server application called Jellyfin. A media server replacement that really aims to compete with the likes of Plex. I have been a Plex user for some time and even have a life time subscription. Plex being so simple to use with such a community it made the choice hard. So why were some reasons I went ahead and deployed a Jellyfin server: 

1. It's Opensource 
2. Full Control to You the Admin 
3. Plugin Selection (Especially LDAP)

For this I am going to be using a spare Dell Precision T5500 Workstation that I have lying around. It will be boasting a single dual core CPU, with 12GB of RAM and 4x300 GB HDD in a RAID 1 configuration. So with enough power under the hood it's time to setup the system. 

### Installing Our Operating System 

The first issue with hosting any application is what operating system to load. Jellyfin has a wide arrangement of support for different operating systems including Windows and Linux. Ultimately for myself I choose Ubuntu as it looked to be where they have a lot of their repository support, and there documentation is very good for this particular Linux distribution. To make it **_even more fun_** we will be setting up the server with the latest and greatest 20.04 release. 

We will be going for a really basic setup. Establish your static IP during install (easiest way but you can use Netplan as well if you want to practice setting an IP using the actual subsystems), install using the entire disk, and really any additional packages we will install so no need to select any additional. As well you should create a new user during installation that is an administrator. This will be important for the SSH hardening we will be doing, and I won't be creating the user manually after installation. Queue 30 minutes for installation... 

### SSH Hardening and Configuration 

*With that* done we can do some basic security for SSH. First always a good practice make a copy of the config file we will be modifying: 

> cp /etc/ssh/sshd_config /etc/ssh/sshd_config 

Before doing any work it's just a wise idea to have a backup in case we mung up our configuration later. Firstly we need to setup some SSH keys on our server. Depending on how you will be accessing your Linux host you might need to go down one of two routes: 

> - [OpenSSH SSH Key Authentication](https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-linux-server)
> - [PuTTY SSH Key Authentication](https://devops.ionos.com/tutorials/use-ssh-keys-with-putty-on-windows/)

For me I did PuTTY as I use a Windows host as my main machine. Though for you Linux or Mac users the first guide will teach you how to use openssh toolset to generate a RSA key and copy it to the remote host. It's important you copy the RSA key with the user you setup during installation. You don't want to copy it to the root account as we are going to be disabling the root login. 

Now that we have the RSA keys generated and added to our accounts authorized_keys file we need to modify the SSH Daemon. To do this we will use our favorite text editor and modify the config file we made a backup of earlier: 

> vim /etc/ssh/sshd_config 

So in this configurtion file we are looking to do three things. Change the default port from 22 (SSH Standard) to a non-standard port. For this we will choose 8822. As well we are going to disable the ability to login with a password via SSH as well as disabling the ability for the root account to login through SSH. Just look for the three directives using the text editors search features of by manually reading the config file... your choice: 

```
Port 8822
...
PermitRootLogin no 
...
PasswordAuthentication no
```

Finally you can restart the SSHD daemon, when we do this our session will be disrupted. So you will need to reaccess the server via SSH using the new port 8822 instead of 22. If you did everything right then you should login via your new key without issue. If there is one you will need to console to the machine and fix what ever was incorrectly done. 

### Local Firewall Using ufw

The last thing we will want to do at a subsystem level is configure our hosts firewall. We will be enabling AD authnetication on the machine, but I will cover that when we integrate Jellyfin to AD. With Ubuntu it comes loaded with a software call **_ufw_**. We want to make sure that we are able to SSH to our server but still able to access the necessary web ports for Jellyfin. To do this we need to issue the following ufw commands: 

```
ufw allow 8822
ufw allow http
ufw allow https
ufw allow 8096
```

The got'cha is we can't see **_ufw_** rule sets without enabling them so if you get them wrong you will be kicked from your session and have to console to the machine. If you are confident you added the right rules you can then enable the firewall: 

> ufw enable

### Lastly Configure the NAS Share and Install Jellyfin 

In order for Jellyfin to play media it has to know about where it is. For my setup I use as NAS that runs CIFS so you will need **_cifs-utils_** if you are going down the same route as me: 

> apt install cifs-utils 

Then we can create a credential file in the root home folder with the following information: 

```
vim /root/nas.smb 
...

user=<share_admin>
password=<admin_password>
```

Then an entry in the **_/etc/fstab_** can be added:

```
//10.0.0.100/Volume_1   /media/nas      cifs    uid=0,credentials=/root/nas.smb        0 0
```

Now with all the base work done we can install Jellyfin. Really this article is about some of the hardships I faced with configuring Jellyfin and the documentation on their site is actually **really great** for just installing the application. So I actually advise you follow these steps to install it on Ubuntu and configure the necessary repositories: 

> [Install Jellyfin on Ubuntu](https://jellyfin.org/docs/general/administration/installing.html#ubuntu)

Once you have the packages you should be able to access the site via http://<host>:8096. From there it will have you create your initial administrator user, as well as setup your libraries. You might immediately find difficulties with meta data and scraping. That is beyond the scope of any of the discussions in this series so unfortunately that is something you will have to hurdle. Though we need to get it secure with a SSL certificate and also tie the application into a central authentication server (which we will be using Microsoft Active Directory for). To review these use the links below: 



- [Jellyfin, Securing with SSL Certificates]({{ site.baseurl }}/Jellyfin-SSL)
- [Jellyfin, Centralizing Authentication with AD]({{ site.baseurl }}/Jellyfin-LDAP)

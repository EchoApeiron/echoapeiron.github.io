---
title: Jellyfin, Centralizing Authentication with AD
author: Dion Pezzimenti
tags: [Networking, Linux, LDAP, Active Directory]
categories: [Jellyfin, Authentication]
date: 2020-10-19 10:00:00 +0800
image: /assets/img/posts/jellyfin.png
---
Onto what is currently the final installment of my Jellyfin server expedition. So far we have done some base server setup, configured the base application, did some networking to enable remote access, and deployed a reverse proxy to do our SSL offloading. Quite a bit, and if you need a review here are the links to the previous blog post:

- [Jellyfin, Setting up the Server]({{ site.baseurl }}/posts/Jellyfin-Initial)
- [Jellyfin, Securing with SSL Certificates]({{ site.baseurl }}/postsJellyfin-SSL)

Now we are going to do one other really cool thing and tie not only the application but our server into our central authentication server. For this we will be using the old trite and true Microsoft Active Directory. One Windows dominates with this technology and two it works very well out of the box with minimal configuration. We may touch on a few things like OUs, Users, and Groups. Beyond that though it will be assumed you have a Domain Controller or some sort of LDAP server already deployed. As we will be using LDAP specifically any LDAP server will be sufficient. You will need to know how to get your own Distinguished Names (or DNs as they are referred to sometimes). 

### Linux and Active Directory 

I remember the first time I attempted to do this... and it was a massive failure back in 2010. While possible it was a difficult to achieve task. Then winbind got very mature and was doing work in conjunction with Samba. Now you will see just how simple it can be to tie  your Linux server to Active Directory. Which means you have no excuse to keep doing it going into the future. 

First we need to install the required packages. The main biggies are going to be realmd, sssd, some PAM modules, and some other supplementary stuff for directory creation and the sorts. You can install the required software running the command below: 

```root@server:~# apt -y install realmd sssd sssd-tools libnss-sss libpam-sss adcli samba-common-bin oddjob oddjob-mkhomedir packagekit```

Since Active Directory is very DNS driven we need to make sure we can communicate with the DNS server that is hosting the zone for our Active Directory. You can do this by using Netplan (which you may have used to configure your static... I didn't for this for sake of ease). For this we just make a small config change to a YAML file in the Netplan configuration you can find it using the ``ls`` command on the `/etc/netplan` directory. Then simply back it up and edit as follows:

```
cp /etc/netplan/01-netcfg.yaml /etc/netplan/01-netcfg.yaml.bkup
vim /etc/netplan/01-netcfg.yaml
...

    nameservers:
      addresses: [192.168.0.100]

netplan apply 
```

One important fact about YAML is that tab characters are illegal. So be sure that you are indenting using only the spacebar or it will break the file. We then need to discover the domain using our `realm` command and should get an output like the following if everything is working properly, and your server can communicate with the Active Directory server's DNS: 

```
root@hinata:/etc/netplan# realm discover domain.com
domain.com
  type: kerberos
  realm-name: DOMAIN.COM
  domain-name: domain.com
  configured: kerberos-member
  server-software: active-directory
  client-software: sssd
  required-package: sssd-tools
  required-package: sssd
  required-package: libnss-sss
  required-package: libpam-sss
  required-package: adcli
  required-package: samba-common-bin
  login-formats: %U
  login-policy: allow-realm-logins
```

With the domain discoverable this means we can then join it. To do so we are going to specify a user that isn't the Administrator account (in my domain it is disabled). If you are using your Administrator account you don't need to specify a user, but otherwise do as I do with the join command below: 

`realm join -U admin_user domain.com`

You will then be prompted for the password. If everything worked you will get no notification which can be daunting but rest assured that's good. If an error is thrown then that's bad. Before we are able to test any users though we have to make some changes to PAM (Pluggable Authentication Module). Make a backup of the config file, but edit the file below as shown and then add the following entry as the very last line of the file: 

```
vim /etc/pam.d/commoon-session
...

session optional        pam_mkhomedir.so skel=/etc/skel umask=077
```

That will make sure home directories are created as our domain users login to the server. Now we can do do a test to make sure we can switch into a domain user. We can use the `su` command to accomplish this as so:

`root@dlp:~# su - domain_user@domain.com`

If everything works correctly you should see the prompt of your CLI change: 

`domain_user@domain.com@server:~$`

One final thing you can do but this is optional is make it to where you can omit the domain name. You do this by modifying the sssd.conf file: 

```vim /etc/sssd/sssd.conf```

You want to find the directive `use_fully_qualified_names` and set it's value to `False`. It is case sensative so you have to type False exactly like that. Just be sure to restart the sssd daemon to make the change take effect: 

`systemctl restart sssd.service`

**_Again all this is optional!_** Even the next part is optional but for those who have used AD you can already see the beauty of being able to have all your logins managed from this central source. But now that we can use our AD users on our Linux system there are some other things you might want to do. Like add certain users to the sudoers group. But for now we are going to move on and focus on integrating Jellyfin in with AD.


### Jellyfin and the LDAP Plugin

The LDAP plugin itself is super simple to install. We just need to access the Plugins menu from the Admin Dashboard in Jellyfin. Then click on repositories and the LDAP plugin should be the first to appear in the respository (at least this was the case for me):

<img src="https://drive.google.com/uc?export=view&id=1iefLZu_xRKfMWSVHqa-TLaapjmBkBguf" alt="Jellyfin Plugins Menu" height="450px" /> 

Once the plugin is installed you will need to restart the `jellyfin` service itself: 

`systemctl restart jellyfin.service` 

From there you have two different approaches to how you can modify the configuration for this. You can either enter the values in the web portal or modify the configuration file located in the following directory (at least for me): 

`/var/lib/jellyfin/plugins/configurations`

From there you can modify the `LDAP-Auth.xml` file. Below is the example of my configuration file, but I will explain what each value means: 

```
<?xml version="1.0"?>
<PluginConfiguration xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema">
  <LdapServer>directory.server.com</LdapServer>
  <LdapBaseDn>DC=server,DC=com</LdapBaseDn>
  <LdapPort>389</LdapPort>
  <LdapSearchAttributes>SamAccountName</LdapSearchAttributes>
  <LdapUsernameAttribute>SamAccountName</LdapUsernameAttribute>
  <LdapSearchFilter>(memberOf=CN=Jellyfin-Users,OU=O-Groups,DC=server,DC=com)</LdapSearchFilter>
  <LdapAdminFilter>(memberOf=CN=Jellyfin-Admins,OU=O-Groups,DC=server,DC=com)</LdapAdminFilter>
  <LdapBindUser>CN=Jellyfin LDAP,OU=Service-Accounts,OU=O-Users,DC=server,DC=com</LdapBindUser>
  <LdapBindPassword>J3llyfin</LdapBindPassword>
  <CreateUsersFromLdap>true</CreateUsersFromLdap>
  <UseSsl>false</UseSsl>
  <UseStartTls>false</UseStartTls>
  <SkipSslVerify>false</SkipSslVerify>
</PluginConfiguration>
```

There is a lot going on in this configuration. Firstly we have these three values that really help to define how we will connect and start searching our domain: 

```
<LdapServer>directory.server.com</LdapServer>
<LdapBaseDn>DC=server,DC=com</LdapBaseDn>
<LdapPort>389</LdapPort>
```

Our _LdapServer_ is the main node we will be using to integrate to our AD environment. As well it will be the one we issue all our LDAP requests to. For our _LdapBaseDn_ it refers to the starting location in the Organizational Units to start searching in the domain. For this we pretty much specify the base domain as the search start. And the _LdapPort_ is just the port that will be used to form the LDAP socket for queries. 

```
<LdapSearchAttributes>SamAccountName</LdapSearchAttributes>
<LdapUsernameAttribute>SamAccountName</LdapUsernameAttribute>
```

These two attributes pretty much state what we are looking for as a value to check against for our authentication. It is a Name Attribute specific to Windows domains. So we will gloss over it as it is still a pretty widely used field. 

```
<LdapSearchFilter>(memberOf=CN=Jellyfin-Users,OU=O-Groups,DC=server,DC=com)</LdapSearchFilter>
<LdapAdminFilter>(memberOf=CN=Jellyfin-Admins,OU=O-Groups,DC=server,DC=com)</LdapAdminFilter>
```

These two filters though are extremely important. These two don't just simply look for Organizational Units in our directory. They go and look for specific User Groups in our directory. Check who the members of these groups are and then enables them to be created in Jellyfin. However I did notice a small glitch. The admin group doesn't seem to work correctly. But you definitely want to utilize the `LdapSearchFilter` as this will dictate the users in your directory that can use the app. The `LdapAdminFilter` is meant to act as a group that dictates who is a Jellyfin admin and who isn't. 

```
<LdapBindUser>CN=Jellyfin LDAP,OU=Service-Accounts,OU=O-Users,DC=server,DC=com</LdapBindUser>
<LdapBindPassword>J3llyfin</LdapBindPassword>
```

These two values are the bread and butter though. The `LdapBindUser` will be the directory user that Jellyfin authenticates with to do LDAP queries in our domain. If there is an account that doesn't have sufficient privleges or if this is not entered correctly then you won't be able to authenticate using your domain at all. Secondly maike sure the `LdapBindPassword` is also set to the correct value. 

The remainder of the configs are just specifiers to tell Jellyfin how to behave in regards to using LDAP. The first being the one we are concerned with. Ensure that `CreateUsersFromLdap` is set to true. The others are related to LDAPS or using LDAP over SSL which we aren't doing. 

So with all that set as long as the configuration saves properly you have entered all the correct values. You will then be able to login to your Jellyfin instance with your domain accounts. Rather than having to create additional local logins for everyone. 

With that Jellyfin is running strong for me, and I hope that these posts have helped you. If there are questions feel free to reach out to me for further assistance. Till then, see you space cowboy...
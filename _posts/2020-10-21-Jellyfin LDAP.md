---
layout: post
title: Jellyfin, Centralizing Authentication with AD
---
< ANOTHER ADD FOR PAGE FORMATTING STAY TUNED >

https://www.server-world.info/en/note?os=Ubuntu_20.04&p=realmd

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
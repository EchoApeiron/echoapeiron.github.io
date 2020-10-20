---
layout: post
title: Jellyfin, Centralizing Authentication with AD
---
< ANOTHER ADD FOR PAGE FORMATTING STAY TUNED >

https://www.server-world.info/en/note?os=Ubuntu_20.04&p=realmd


<?xml version="1.0"?>
<PluginConfiguration xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema">
  <LdapServer>itachi.digiecho.xyz</LdapServer>
  <LdapBaseDn>DC=digiecho,DC=xyz</LdapBaseDn>
  <LdapPort>389</LdapPort>
  <LdapSearchAttributes>SamAccountName</LdapSearchAttributes>
  <LdapUsernameAttribute>SamAccountName</LdapUsernameAttribute>
  <LdapSearchFilter>(memberOf=CN=Jellyfin-Users,OU=O-Groups,DC=digiecho,DC=xyz)</LdapSearchFilter>
  <LdapAdminFilter>(memberOf=CN=Jellyfin-Admins,OU=O-Groups,DC=digiecho,DC=xyz)</LdapAdminFilter>
  <LdapBindUser>CN=Jellyfin LDAP,OU=Service-Accounts,OU=O-Users,DC=digiecho,DC=xyz</LdapBindUser>
  <LdapBindPassword>J3llyfin</LdapBindPassword>
  <CreateUsersFromLdap>true</CreateUsersFromLdap>
  <UseSsl>false</UseSsl>
  <UseStartTls>false</UseStartTls>
  <SkipSslVerify>false</SkipSslVerify>
</PluginConfiguration>
---
title: OpenSSL, Verifying SSL
author: Dion Pezzimenti
tags: [Networking, Security]
categories: [SSL, Certificates]
date: 2020-11-09 12:30:00 +0800
---
When writing a previous article Jellyfin, Networking and SSL I had a bit of an issue. This issue was grave to the extent that it didn't allow the web app to connect through the reverse proxy. The below link will allow you to re-read if needed, but ultimately it was an issue to how the SSL certificate was implemented. 

[Jellyfin, Securing with SSL Certificates]({{ site.baseurl }}/posts/Jellyfin-SSL)

Ultimately it ended up being a misconfiguration from my part and how my registrar provided the certificates. How did I find out that my SSL certificate was implemented improperly to begin with? Well I was informed that actually we can use the `s_client` feature of `openssl` to validate if a SSL connection is valid or not. So my goal here is to quickly explain how to use `openssl` in order to validate the certificate. As well as how to look up your registrar's direction on installing their SSL certificates. 

### Validate the SSL Cert is Serving Correctly

Since we are hosting on a public DNS and in this instance we assume it is `media.domain.com`. We need a system where `openssl` is available. Linux is a good call but a Mac would actually be awesome here as well. If you are on Windows the best option would be Cygwin which can be installed using the link below: 

[Cygwin Installer](https://www.cygwin.com/install.html)

Once you have a platform that is valid we need to issue the following command to our domain (and port if you are using a different port) to validate the SSL certificate.

`openssl s_client -connect media.domain.com:443`

Ultimately we are looking for the below: 

```
Start Time: 1604962023
Timeout   : 7200 (sec)
Verify return code: 0 (ok)
Extended master secret: no
Max Early Data: 0
```

If we see that Verify return code is 0 then we know that the certificate is applied successfully. However in my instance and in many other instances you may not get a good return code. So what might have gone wrong? Well for me I used Namecheap to acquire my certificate. So I did a simple Google search for *Namecheap Install SSL Certificate*. It took me to the exact article I needed. 

In this instance I need to concantante my server's certificate PEM file with the Certificate Authority Chain provided. Which can be reviewed here: 

[Combine PEM Certificates](https://support.vidyocloud.com/hc/en-us/articles/115000460374-Combining-root-and-intermediate-certificates)

However there may be many different issues related to the issue you are experiencing with your certificate. It's always a good idea to validate anyways to enusre your SSL is working properly. If it's not the data may not be encrypted properly for your needs. 

### Conclussion 

The most important thing to take away from this post albeit it's short. Is always validate your SSL connection. It can cause a lot of adverse side effects to your application if you are not careful. As well always seek consultation from your registrar if you have issues. 
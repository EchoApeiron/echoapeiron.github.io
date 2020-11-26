---
title: Jellyfin, Networking and SSL
author: Dion Pezzimenti
tags: [Networking, Linux, Security, Web Hosting]
categories: [Jellyfin, Securing]
date: 2020-10-19 10:00:00 +0800
image: /assets/img/posts/jellyfin.png
---
In my last entry I talked about the initial server setup for my Jellyfin server. Some of the basic things I do whenever setting up a Linux host then getting the application installed. If you haven't read that and need to please see the link below: 

[Jellyfin, Setting up the Server]({{ site.baseurl }}/posts/Jellyfin-Initial)

Where we left off you were at the point where you were configuring the application. Jellyfin runs off port 8096 by default for it's base traffic. We will need this port as I haven't gotten the app to use the reverse proxy we are going to build. But for our web browser we will be accessing the site via port 80 and port 443. 

### Why am I mentioning these ports? 

Because the beautiful thing about Jellyfin is that it can do remote access out of the box without a license (_suck it Plex and Emby_). We just need to do some port forwarding on our router to allow us to access our site through the public IP that our ISP provides us. 

There are a myraid of ways to do this depending on the router or service provider you have. So below are just some articles discussing NAT, PAT, and Port Forwarding techniques (this is alot but should encompass all the things that are going on here). 

> - [Network Address Translation FAQ - Cisco](https://www.cisco.com/c/en/us/support/docs/ip/network-address-translation-nat/26704-nat-faq-00.html)
> - [Difference Between NAT and PAT - GeeksforGeeks](https://www.geeksforgeeks.org/difference-between-network-address-translation-nat-and-port-address-translation-pat/)
> - [How to Forward a Port](https://portforward.com/)

As well you might need some help finding your Public IP address that your ISP provided you. You can Google 'what is my IP' or use the link below:

> [What is My IP Address](https://whatismyipaddress.com/)

Finally this step I will talk more about as this is the process I used. But you may find you want to lease a domain. I highly suggest doing this as we will need to purchase an SSL certificate to complete this project. No SSL service can run on Linux (or any system really) without a valid certificate. It's just the way it is. 

I suggest using Namecheap as the name implies, they have some pretty good rates for their services. Their customer service can be less than desired. All and all it isn't the worst company for the price. Their article below details how you can request a domain from them: 

> [How to Buy a Domain Name - NameCheap](https://www.namecheap.com/blog/how-to-buy-a-domain-name-dp/)

For now you can just setup the initial domain. Even add a DNS host entry for your media server now as you will need this for the Certificate Signing Request or CSR we are about to submit. I made mine `home.domain.com`, but here are some examples and combinations you could use: 

> - jellyfin.domain.com
> - jellyfin.domain.xyz
> - jellyfin.domain.net
> - media.domain.com
> - stream.domain.com 

As you can see that we can use pretty much anything for a subdomain but depending on the Top Level Domain or TLD will depend on our price. Just ensure you nab a domain and add a DNS pointing something to the public IP address you received before. 

**_And now_** with those prep steps out of the way we can start doing a little server stuff. The first thing we need to do is generate the CSR I was mentioning before. This is going to do two things: 

1. Firstly it's going to create the Certificate Signing Request which we need to provied to the Certificate Authority or we can't get our cert. 
2. It's going to create our certificate's private key. **THIS IS SUPER IMPORTANT DON'T LOSE THIS OR GIVE IT OUT OR YOU MUST REQUEST A NEW CERTIFICATE** 

In order to do this we are going to use OpenSSL issuing the following command: 

```openssl req -newkey rsa:2048 -keyout PRIVATEKEY.key -out MYCSR.csr```

You can name your PRIVATEKEY.key anything, but keep the extension .key. As for the CSR you really only need to copy the contents of this. You can use these contents to request your certificate through NameCheap. Once you have bought one you can use their article for instructions how to activate it with the CSR we just generated:

[Activate SSL Certificate - NameCheap](https://www.namecheap.com/support/knowledgebase/article.aspx/794/67/how-do-i-activate-an-ssl-certificate)

This can be a bit of a long winded process but in the end after you have done the domain validation and everything the certificate should be sent to the email you provided. So all you need to do is copy them to the server. For now you can keep the cert file (the CRT file) and the bundle file (or authority chain but should be named ca-bundle) with the private key. 

This next part we are going to use `nginx` to reverse proxy and do the SSL termination for Jellyfin. Firstly we need to install the `nginx` software on the server. This is really easy all we need to do is update our apt repository (which probably has already been done), and then install the package: 

```
apt update
apt install nginx 
```

We can then create a folder in `nginx` config folder to store our certificates using the mkdir command: 

```mkdir -p /etc/nginx/ssl/<domain>```

Then you can copy your cert files right to here. You can verify the files have the proper permissions as the other install files for nginx using `ls -la`. It would be best to compare the cert against maybe the main config file like so: 

```
root@server:~# ls -la /etc/nginx/nginx.conf
-rw-r--r-- 1 root root 1490 Oct 16 07:59 /etc/nginx/nginx.conf
root@server:~# ls -la /etc/nginx/ssl/home/
total 48
drwxr-xr-x 2 root root 4096 Oct 17 00:35 .
drwxr-xr-x 3 root root 4096 Oct 16 08:33 ..
-rw-r--r-- 1 root root 4135 Oct 16 08:50 chain.pem
-rw-r--r-- 1 root root 1679 Oct 16 07:56 server.key
-rw-r--r-- 1 root root 2065 Oct 16 08:50 server.pem

```

You might notice that mine are named a bit differently. Your certs most likely came in a plain text form which actually means they are in a PEM format (see below for more information, it is a ServerFault question but very detailed). So I just renamed my certificate chain file and my public certficate to a pem keeping my private key as a .key file to distinguish them: 

[What is a PEM File - Question from ServerFault](https://serverfault.com/questions/9708/what-is-a-pem-file-and-how-does-it-differ-from-other-openssl-generated-key-file)

Before we are able to use the certificate files though. Since we got them from name cheap we will have to concatenate (join together essentially) the chain and the server.pem file. We can do this simply but using the `cat` command in Linux:

```cat chain.pem server.pem > full_chain.pem```

When we list the contents make sure they have the same permissions and everything as the other files:

```
root@server:~# ls -la /etc/nginx/nginx.conf
-rw-r--r-- 1 root root 1490 Oct 16 07:59 /etc/nginx/nginx.conf
root@server:~# ls -la /etc/nginx/ssl/home/
total 48
drwxr-xr-x 2 root root 4096 Oct 17 00:35 .
drwxr-xr-x 3 root root 4096 Oct 16 08:33 ..
-rw-r--r-- 1 root root 4135 Oct 16 08:50 chain.pem
-rw-r--r-- 1 root root 1679 Oct 17 00:35 full_chain.pem
-rw-r--r-- 1 root root 1679 Oct 16 07:56 server.key
-rw-r--r-- 1 root root 2065 Oct 16 08:50 server.pem

```

Finally we just need to tell nginx how to proxy our request but also specify the certificate information and to use SSL in order have a secure connection. Create the following file then add a configuration like the example provided using your relavant server information: 

```vim /etc/nginx/conf.d/reverse-proxy.conf```

```
upstream backends {
    server 192.168.0.245:8096;
}

server {
    listen 80;
    server_name <server_url>; 
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;  
    server_name <server_url>;
    allow all; 

    ssl_certificate /etc/nginx/ssl/home/full_chain.pem;
    ssl_certificate_key /etc/nginx/ssl/home/server.key;

    location / {
        proxy_pass http://backends;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-Protocol $scheme;
        proxy_set_header X-Forwarded-Host $http_host;
    }
}
```

Let's take a quick moment to break down the important stuff, most of this is just standard HTTP header stuff, but there is some things that you should be aware of so you know how it works: 

```
upstream backends {
    server 192.168.0.245:8096;
}
```

This defines a variable effectively called `backends`. We tell `nginx` here that we have a host that has an IP address of 192.168.0.245 and will be listening on Port 8096 for our requests. 

```
server {
    listen 80;
    server_name <server_url>; 
    return 301 https://$host$request_uri;
}
```

This defines effectively what could be considered our first virtual directory. It has `nginx` listen on port 80 for any requests to our server URL. It does one more thing where it will also issue a HTTP 301:

[HTTP 301 - Mozilla](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/301)

This is the HTTP server or `nginx` in our case that the page that was hosted on port 80 has been moved. It also specifies the URI string that should be connected to now instead: 

```
server {
    listen 443 ssl;  
    server_name <server_url>;
    allow all; 

    ssl_certificate /etc/nginx/ssl/home/full_chain.pem;
    ssl_certificate_key /etc/nginx/ssl/home/server.key;
```

With our previous declaration we said that we moved the page to effectively the same URI string but on Port 443 or HTTPS. So we make another declaration of a server listening on port 443. However we specify we allow all connections and point to the locations of certificates so that the service runs proper.

```
    location / {
        proxy_pass http://backends;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-Protocol $scheme;
        proxy_set_header X-Forwarded-Host $http_host;
    }
}
```

Then lastly we tell what server URI's that `nginx` is proxying for and specifies it to connect to the backends we defined earlier. It also creates some variables with some additional header information that will be added to the packet to make sure the proxying continues to work. 

Finally we just restart the service and all should be setup and ready to go:

```systemctl restart nginx```

Now from any web browser or the web app you should be able to use the domain you setup and be redirected to a secure HTTPS page. However it is a proxied connection to your Jellyfin server. 

There is only one more exploit in this adventure and that will be centralizing the login. If you still have yet to setup a Jellyfin server I will provide the link the initial setup again down below: 

- [Jellyfin, Setting up the Server]({{ site.baseurl }}/posts/Jellyfin-Initial)
- [Jellyfin, Centralizing Authentication with AD]({{ site.baseurl }}/posts/Jellyfin-LDAP)
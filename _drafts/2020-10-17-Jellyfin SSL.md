---
layout: post
title: Jellyfin, Securing with SSL Certificates
---
    upstream backends {
        server 10.0.0.245:8096;
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
    
        ssl_certificate /etc/nginx/ssl/home/server.pem;
        ssl_certificate_key /etc/nginx/ssl/home/server.key;
        ssl_trusted_certificate /etc/nginx/ssl/home/chain.pem;

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

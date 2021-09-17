---
title: Deploy Your Own Password Manager
date: 2021-09-17 14:45:49
tags:
    - Docker
    - Bitwarden
    - 1Password
    - Tutorial
---

I have tried many password mangers like Google Chrome built-in, iCloud Keychain, LastPass, 1Password. Each of them has different advantages in different environments. For example, iCloud Keychain works perfectly in all Apple Device. 1Password is such thoughtful product. However, each product has little defect like cross-platform problem or price ~~That may be my defect ;-)~~.



I found many Open Source alternatives which can be hosted and controlled by myself. The most important thing is most of them can cover all my platforms. I picked up **Bitwarden** as my password manager solution. And I tried to deploy it on my Oracle Always Free server. 



It is quite easy if you choose to host it with docker.

<!--more-->



## Install Docker

You can basically follow the [offical installation tutorials](https://docs.docker.com/engine/install/).



*Maybe you will get an error when you are testing your docker*

> Got permission denied ... /var/run/docker.sock: connect: permission denied

This is about your docker daemon's permission. There are many [solutions](https://docs.docker.com/engine/install/linux-postinstall/) for this. Choose one that suit yourself!



You shold also install [docker-compose](https://docs.docker.com/compose/install/)



## Make Some Preparations

We need some configuration files for Bitwarden.



-  docker-compose.yml

  ```yaml
  version: '3'
  
  services:
    db:
      image: postgres:latest
      container_name: postgres_db
      user: 1001:1001 # change this to fit your UID/GID
      restart: always
      environment: 
        - POSTGRES_USER=vaultwarden
        - POSTGRES_PASSWORD=vaultwardenpgadmin
      volumes: 
        - ./db-data:/var/lib/postgresql/data
  
    vaultwarden:
      image: vaultwarden/server:latest
      container_name: vaultwarden
      restart: always
      environment:
        - DATABASE_URL=postgresql://vaultwarden:vaultwardenpgadmin@db:5432/vaultwarden
        - WEBSOCKET_ENABLED=true  # Enable WebSocket notifications.
        - LOG_FILE=/data/vaultwarden.log
        - SIGNUPS_ALLOWED=true
        - INVITATIONS_ALLOWED=false
      volumes:
        - ./vw-data:/data
  
    caddy:
      image: caddy:2
      container_name: caddy
      restart: always
      ports:
        - 80:80  # Needed for the ACME HTTP-01 challenge.
        - 443:443
      volumes:
        - ./Caddyfile:/etc/caddy/Caddyfile:ro
        - ./caddy-config:/config
        - ./caddy-data:/data
      environment:
        - DOMAIN=  # Your domain, prefixed with http or https.
        - EMAIL=   # The email address to use for ACME registration.
        - LOG_FILE=/data/access.log
  ```

- Caddyfile

  ```
  {$DOMAIN}:443 {
    log {
      level INFO
      output file {$LOG_FILE} {
        roll_size 10MB
        roll_keep 10
      }
    }
  
    # Use the ACME HTTP-01 challenge to get a cert for the configured domain.
    tls {$EMAIL}
    #tls FILE.pem FILE.key.pem
  
    # This setting may have compatibility issues with some browsers
    # (e.g., attachment downloading on Firefox). Try disabling this
    # if you encounter issues.
    encode gzip
  
    # Notifications redirected to the WebSocket server
    reverse_proxy /notifications/hub vaultwarden:3012
  
    # Proxy everything else to Rocket
    reverse_proxy vaultwarden:80 {
         # Send the true remote IP to Rocket, so that vaultwarden can put this in the
         # log, so that fail2ban can ban the correct IP.
         header_up X-Real-IP {remote_host}
    }
  }
  
  ```

  If you don't want Caddy server automatically get a SSL certificate, change that *tls*.

  

  
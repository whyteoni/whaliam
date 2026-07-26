+++
date = '2026-07-25T07:26:58-07:00'
draft = false
title = 'Using SearXNG as You Default Search'
tags = ["search","homelab","project-majel"]
category = "project"
toc = true
+++

[SearXNG](https://github.com/searxng/searxng) is a meta search engine that you run locally. It provides very fast results without a lot of fluff and tracking. You can fine-tune it to only use certain sources, search certain material types, skip social media, or even exclusively search social media. It is a fantastic tool and something anyone with the inclination and resources should consider setting up.

## Setting up the Service

My setup is a lot more complicated than it absolutely has to be. But, you wouldn't be here if you wanted the generic guide.

### The Container

I'm using a container and the [podman](https://podman.io/) runtime (a Docker alternative that has several advantages and is my go to solution). I won't go into details about getting your container runtime functional in this guide, though I plan to at some point in the future show how I have my setup. I am also running [podman-compose](https://docs.podman.io/en/latest/markdown/podman-compose.1.html) to orchestrate my containers and help make management easier. Here's what that looks like:

```yaml
  searxng-core:
    container_name: searxng-core
    image: searxng/searxng:latest
    restart: always
    ports:
      - "8080:8080"
    volumes:
      - ./volumes/searxng/core-config/:/etc/searxng/:Z
      - ./volumes/searxng/core-data:/var/cache/searxng/
    networks:
      - majel

  searxng-valkey:
    container_name: searxng-valkey
    image: valkey/valkey:9-alpine
    command: valkey-server --save 30 1 --loglevel warning
    restart: always
    volumes:
      - ./volumes/searxng/valkey-data:/data/
    networks:
      - majel

networks:
  majel: {}
```

In my case I am standardizing my compose file to use directly mounted folders, instead of runtime administered volumes. I have trust issues.

Also, I am explicitly using the `majel` network to keep all [Project: Majel](/tags/project-majel) components together. I am not 100% certain this is necessary, but it works so I have not tested it further.

## Accessing SearXNG

You can simply access this service now by navigating to `http://localhost:8080` and running your search. For other systems on your network, assuming you open the correct firewall ports, they would go to `http://<your IP address>:8080`. I am particular, and would not accept something so crude, especially knowing it would be a turn off to other denizens of my network (read: _family members_).

### DNSMasq

I will not go into all the arcane things I did to make this service work. Suffice it to say, for what lies ahead you need to have your local DNS point at your server, which should have either a static address, or at least a way to dynamically update DNS. It is for this reason I chose to use [DNSMasq](https://wiki.archlinux.org/title/Dnsmasq), though this Faustian bargain has taken many hours from my life.

### Nginx Proxy

Finally, I created an Nginx proxy with a self signed certificate. In combination with the above DNS configuration I am able to use human-friendly URLs. The `servername` passed to the proxy is used to direct the traffic internally. This also means I do not need to expose those ports to the rest of my network. Everyone wins.

I am using this concept with all of my local services, and keep each endpoint/server as a separate config file under `sites-available`. Here's my config for SearXNG:

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name searxng.local;

    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name searxng.local;

    ssl_certificate     /etc/nginx/cert/local.crt;
    ssl_certificate_key /etc/nginx/cert/local.key;

    location / {
            proxy_pass http://localhost:8080;

            proxy_set_header    Host    $host;
            proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header    X-Forwarded-Proto       $scheme;
    }
}
```

## Setting Your Default Search Engine

In Firefox, and I assume something very similar for other browsers, go to `Settings` -> `Search` -> `Add Search Engine`. Enter in the following information:

- Name: `SearXNG`
- URL: `https://searxng.local/search?q=%s`
- Keyword: `@xng`

I decided to make this my default search engine and have not looked back.

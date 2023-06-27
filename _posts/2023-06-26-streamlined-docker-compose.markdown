---
layout: post
title:  "streamlined docker compose"
date:   2023-06-26 17:01:57 +0000
categories: docker lemmy
---
I apologize for the wall of text, but I wanted to thoroughly explain my choices and reasoning for this docker-compose.yml. I hope you find it useful.

I find the docker compose provided by the Lemmy devs to be overly complicated. I have pared it down to the essentials to get it running nicely with your choice of reverse proxy. In this example I use Nginx Proxy Manager (NPM) which is very simple to use and I recommend it to anyone who just wants a simple Lemmy install with easy automation of SSL certs. If you want to use something other than NPM (Caddy, Traefik, Nginx, etc) you can just omit the section for `nginxproxymanager:` and insert your own reverse proxy preference.

It should be noted that I have intentionally removed postfix from the docker-compose.yml as I believe that it's a bad idea for people to run postfix without knowing what they are doing. If you run postfix from your home server or from a VPS you will almost certainly have most/all of your outgoing mail rejected by recipient servers due to blacklisting. I recommend using one of the many mail services such as sendgrid, mailgun, postmark, etc. If you feel differently, feel free to do it your way.

This simplified compose removes the necessity to have separate docker networks and only requires you to expose the port(s) of your reverse proxy of choice. All other internal network communication between containers in this compose are done via on the default network that is automatically created when you do `docker compose up -d`.

You only need to reverse proxy to the lemmy_ui on port 1234 and to lemmy on port 8536 for /.well-known, /feeds, /pictrs, and /api. This is done in different ways depending on which product you use for a reverse proxy. [I wrote a short guide for NPM]({% post_url 2023-06-26-proxy-npm-setup %}), you are on your own if you use a different product.

Please be aware that the logging is reduced to only log on ERROR instead of the default setting which is WARN. This is done to keep the log files from being flooded by innocuous warnings that may or may not indicate trouble on your instance. Lemmy logging is very chatty and I wanted to quiet it down after I was sure everything was working ok. You may want to change your logging back to WARN temporarily if you are having troubles and can't figure out what's going wrong. Just replace every instance of the word "error" with "warn". I also have made settings in my systems overall Docker logging to reduce log storage in general for ALL containers. If you want more information on that find me (tet42) on the [Lemmy Admins Matrix chat](https://matrix.to/#/#lemmy-support-general:discuss.online){:target="_blank"}.

The lemmy.hjson referenced to in this compose can be obtained from the official GitHub of the Lemmy devs. [https://github.com/LemmyNet/lemmy/blob/main/config/defaults.hjson](https://github.com/LemmyNet/lemmy/blob/main/config/defaults.hjson){:target="_blank"} If you use theirs be sure to either name it lemmy.hjson or change the filename in your docker-compose.yml to defaults.hjson and be sure to edit the file to fit your specific needs.

Alternately you can refer to the example [lemmy.hjson]({% post_url 2023-06-26-lemmy-hjson %}) file that I have provided and is known to work with my examples. You will still need to change some things, but it is a little more straight forward.

You may want to edit/change the pict-rs settings to better suit your needs. There are also adiitional Postgres settings you can make to optimize your database. Both are more advanced settings and are outside the scope of this document.

Please note that I am pulling Lemmy version 0.18 `dessalines/lemmy:0.18 and dessalines/lemmy-ui:0.18` and pict-rs 0.4.0-rc.7 `asonix/pictrs:0.4.0-rc.7` in this example docker-compose.yml. You may want to change those version tags to whatever version you intend to run for your instance. When Lemmy is upgraded in the future, you will need to edit your docker-compose.yml, increment the versions to match the newer version you want to run, and then run `docker compose up -d` to automatically pull the newer images and restart the necessary containers. Version tags can be viewed on the [LEMMY DOCKERHUB PAGE](https://hub.docker.com/r/dessalines/lemmy/tags){:target="_blank"}. pict-rs can also be upgraded at your discretion, their version tags can be viewed on the [PICT-RS DOCKERHUB PAGE](https://hub.docker.com/r/asonix/pictrs/tags){:target="_blank"}


{% highlight ruby %}
version: "3.3"
services:
  nginxproxymanager:
   container_name: nginxproxymanager
   image: 'jc21/nginx-proxy-manager:latest'
   restart: always
   ports:
     - '80:80'
     - '81:81'
     - '443:443'
   volumes:
     - ./nginxproxymanager/data:/data
     - ./nginxproxymanager/letsencrypt:/etc/letsencrypt
     - ./nginxproxymanager/logs:/data/logs

  lemmy:
    image: dessalines/lemmy:0.18
    container_name: lemmy
    hostname: lemmy
    restart: always
    environment:
      - RUST_LOG="error,lemmy_server=error,lemmy_api=error,lemmy_api_common=error,lemmy_api_crud=error,lemmy_apub=error,lemmy_db_schema=error,lemmy_db_views=error,lemmy_db_views_actor=error,lemmy_db_views_moderator=error,lemmy_routes=error,lemmy_utils=error,lemmy_websocket=error"
    volumes:
      - ./lemmy.hjson:/config/config.hjson
    depends_on:
      - postgres
      - pictrs

  lemmy-ui:
    image: dessalines/lemmy-ui:0.18
    container_name: lemmy_ui
    environment:
      - LEMMY_UI_LEMMY_INTERNAL_HOST=lemmy:8536
      - LEMMY_UI_LEMMY_EXTERNAL_HOST=localhost:1234
      - LEMMY_HTTPS=false
      - LEMMY_UI_EXTRA_THEMES_FOLDER=/lemmy-ui/extra_themes
    volumes:
      - ./volumes/lemmy-ui:/lemmy-ui
    depends_on:
      - lemmy
    restart: always

  pictrs:
    image: asonix/pictrs:0.4.0-rc.7
    container_name: lemmy_pictrs
    hostname: pictrs
    environment:
      - RUST_LOG=error
      - PICTRS__MEDIA__VIDEO_CODEC=vp9
      - PICTRS__MEDIA__GIF__MAX_WIDTH=256
      - PICTRS__MEDIA__GIF__MAX_HEIGHT=256
      - PICTRS__MEDIA__GIF__MAX_AREA=65536
      - PICTRS__MEDIA__GIF__MAX_FRAME_COUNT=400
    user: 991:991
    volumes:
      - ./volumes/pictrs:/mnt:Z
    restart: always
    deploy:
      resources:
        limits:
          memory: 690m

  postgres:
    image: postgres:15-alpine
    hostname: postgres
    container_name: lemmy_postgres
    environment:
      - POSTGRES_USER=changeme
      - POSTGRES_PASSWORD=changeme
      - POSTGRES_DB=changeme
    volumes:
      - ./volumes/postgres:/var/lib/postgresql/data
    restart: always
{% endhighlight %}
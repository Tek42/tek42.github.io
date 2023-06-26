---
layout: post
title:  "Lemmy NPM Setup Instructions"
date:   2023-06-26 17:01:57 +0000
categories: proxy
---

All settings assume you already have ports 80 and 443 open on your server. You will also need port 81 open to access the NPM admin gui however I highly recommend that you only open this port to the IP you intend to access it from. There is username/password authentication on NPM admin but blocking it from everyone except your IP is an additional layer of security.

Access NPM at http://your.server.ip:81

Default Admin User:

Email:    admin@example.com
Password: changeme

Immediately after logging in with this default user you will be asked to modify your details and change your password.

After you have created your admin login you will be taken back to the dashboard. Click Proxy Hosts, then click Add Proxy Host.

Details tab. Enter your desired instance hostname and hit enter. Then fill in the remaining fields as shown. If you are running any version of Lemmy belov 0.18 you will also need to enable "Websockets Support".

![NPM details page](/assets/details.png)

Custom locations tab, you will want to add entries for /api (as shown) but also add additional entries for /.well-known, /pictrs, and /feeds. All should point to the same hostname and port (lemmy:8536)

![NPM custom locations tab](/assets/custom_locations.png)

SSL tab, select these options and fill in your email address agree then click save. It will automatically request a Letsencrypt cert for you and apply it to your site.

![NPM SSL tab](/assets/ssl.png)

Advanced tab

![NPM Advanced tab](/assets/advanced.png)

In the Advanced tab, paste this in and save.

{% highlight ruby %}
location / {

      # The default ports:
      # lemmy_ui_port: 1235(not used here)
      # lemmy_port: 8536
      # The cooler port:
      # lemmy_ui_port: 1236

      set $proxpass "http://lemmy_ui:1234";
      if ($http_accept ~ "^application/.*$") {
        set $proxpass "http://lemmy:8536";
      }
      if ($request_method = POST) {
        set $proxpass "http://lemmy:8536";
      }
      proxy_pass $proxpass;

      rewrite ^(.+)/+$ $1 permanent;

      # Send actual client IP upstream
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header Host $host;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
{% endhighlight %}
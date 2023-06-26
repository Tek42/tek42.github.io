---
layout: post
title:  "lemmy.hjson"
date:   2023-06-26 17:01:57 +0000
categories: lemmy
---
Here is an example lemmy.hjson that you can use in your Lemmy deployment.

{% highlight ruby %}
{
  # for more info about the config, check out the documentation
  # https://join-lemmy.org/docs/en/administration/configuration.html
  # only few config options are covered in this example config

  setup: {
    # username for the admin user
    admin_username: "changeme"
    # password for the admin user
    admin_password: "changeme"
    # name of the site (can be changed later)
    site_name: "Your Site Name"
  }

  # the domain name of your instance (eg "lemmy.ml")
  hostname: "your.lemmy.url"
  # address where lemmy should listen for incoming requests
  bind: "0.0.0.0"
  # port where lemmy should listen for incoming requests
  port: 8536
  # Whether the site is available over TLS. Needs to be true for federation to work.
  tls_enabled: true

  # pictrs host
  pictrs: {
    url: "http://pictrs:8080/"
  }

  # settings related to the postgresql database
  database: {
    # name of the postgres database for lemmy. This should match whats in your docker-compsoe.yml
    database: "changeme"
    # username to connect to postgres. This should match whats in your docker-compsoe.yml
    user: "changeme"
    # password to connect to postgres. This should match whats in your docker-compsoe.yml
    password: "changeme"
    # host where postgres is running
    host: "postgres"
    # port where postgres can be accessed
    port: 5432
    # maximum number of active sql connections
    pool_size: 5
  }
  
 # Email sending configuration. All options except login/password are mandatory
  email: {
    # Hostname and port of the smtp server
    smtp_server: "your.smtp.server:587"
    # Login name for smtp server
    smtp_login: "changeme"
    # Password to login to the smtp server
    smtp_password: "changeme"
    # Address to send emails from, eg "noreply@your-instance.com"
    smtp_from_address: "your@reply.address"
    # Whether or not smtp connections should use tls. Can be none, tls, or starttls
    tls_type: "starttls"
  }
}
{% endhighlight %}
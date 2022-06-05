---
layout: article
title: Puppet and Letsencrypt
excerpt: >
  Simple(ish) way to get letsencrypt certs and set up auto renewals.
published: true
last_modified_at: 2022-05-15 19:00:00 +0800
tags:
  - 'Configuration Management'
  - 'Operations'
  - 'Automation'
---
### Update 2022-05-15
In most cases you will want to let certbot handle cron and disable the "manage_cron" setting. In the case of Ubuntu, the cron is  handled by a systemd timer.

---

Using the [puppet-nginx](https://forge.puppet.com/modules/puppetlabs/nginx) and [puppet-letsencrypt](https://forge.puppet.com/modules/puppet/letsencrypt) modules, you can automate letsencrypt certificate requests and renewals.


The hiera and puppet class code below sets up nginx with the two domains and redirects everything except the letsencrypt challenge URLs to https. It also serves the challenge files in a separate directory to keep your web root cleaner and easier to manage.


You could add both domains to the one certificate but I've separated them out here as this was one of the things I was testing in this experiment. This just means that the two certificates will get renewed one after the other and nginx will get reloaded after each successful renewal ie. two separate certbot runs, each running their "deploy_hook_commands" if and when they are successfully renewed.

```yaml
---
classes:
  - letsencrypt
  - nginx
  - andrewr::letsencrypt

letsencrypt::email: '------@gmail.com'
letsencrypt::package_ensure: latest

# my own custom module class that takes the hiera and turns it into resources
andrewr::letsencrypt:
  certs:
    'test.andrewrowe.dev':
      certonly:
        'test.andrewrowe.dev':
          ensure: present
          domains:
            - 'test.andrewrowe.dev'
          plugin: 'webroot'
          webroot_paths:
            - '/var/www/letsencrypt/'
#          manage_cron: true
          manage_cron: false # disable for Ubuntu
#          cron_hour: 0
#          suppress_cron_output: true
          deploy_hook_commands:
            - '/bin/systemctl reload nginx.service'
    'test1.andrewrowe.dev':
      certonly:
        'test1.andrewrowe.dev':
          ensure: present
          domains:
            - 'test1.andrewrowe.dev'
          plugin: 'webroot'
          webroot_paths:
            - '/var/www/letsencrypt/'
          manage_cron: true
          cron_hour: 0
          suppress_cron_output: true
          deploy_hook_commands:
            - '/bin/systemctl reload nginx.service'

nginx::ssl_protocols: 'TLSv1.2 TLSv1.3'
nginx::ssl_ciphers: 'TLS13-CHACHA20-POLY1305-SHA256:TLS13-AES-256-GCM-SHA384:TLS13-AES-128-GCM-SHA256:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA:ECDHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA256:DHE-RSA-AES256-SHA:ECDHE-ECDSA-DES-CBC3-SHA:ECDHE-RSA-DES-CBC3-SHA:EDH-RSA-DES-CBC3-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:DES-CBC3-SHA:!DSS'

nginx::nginx_servers:
  'test.andrewrowe.dev':
    ensure: present
    ssl: true
    listen_port: 80
    ssl_port: 443
    ssl_redirect: false
    use_default_location: false
    www_root: '/var/www/test.andrewrowe.dev'
    ssl_cert: "/etc/letsencrypt/live/test.andrewrowe.dev/fullchain.pem"
    ssl_key: "/etc/letsencrypt/live/test.andrewrowe.dev/privkey.pem"
  'test1.andrewrowe.dev':
    ensure: present
    ssl: true
    listen_port: 80
    ssl_port: 443
    ssl_redirect: false
    use_default_location: false
    www_root: '/var/www/test.andrewrowe.dev'
    ssl_cert: "/etc/letsencrypt/live/test1.andrewrowe.dev/fullchain.pem"
    ssl_key: "/etc/letsencrypt/live/test1.andrewrowe.dev/privkey.pem"
nginx::nginx_locations:
  'test.andrewrowe.dev letsencrypt':
    server: 'test.andrewrowe.dev'
    location: '/.well-known/acme-challenge/'
    www_root: '/var/www/letsencrypt'
  'test.andrewrowe.dev root':
    server: 'test.andrewrowe.dev'
    location: '/'
    location_cfg_prepend:
      rewrite: '^ https://$host$request_uri redirect'
  'test1.andrewrowe.dev letsencrypt':
    server: 'test1.andrewrowe.dev'
    location: '/.well-known/acme-challenge/'
    www_root: '/var/www/letsencrypt'
  'test1.andrewrowe.dev root':
    server: 'test1.andrewrowe.dev'
    location: '/'
    location_cfg_prepend:
      rewrite: '^ https://$host$request_uri redirect'
```

The following class allows me to specify the letsencrypt certs all in hiera so there is one place to look for the server setup.

```ruby
# @summary A short summary of the purpose of this class
#
# A description of what this class does
#
# @example
#   include andrewr::letsencrypt
class andrewr::letsencrypt {
  $letsencrypt = lookup({
    'name'  => 'andrewr::letsencrypt',
    'merge' => {
      'strategy' => 'deep',
    },
    default_value => {}
  })

  exec { 'create letsencript www_root':
    path    => $::path,
    command => 'mkdir -p /var/www/letsencrypt',
    creates => '/var/www/letsencrypt'
  }

  if ( $letsencrypt['certs'] ) {
    each($letsencrypt['certs']) |$key, $cert| {
      if ($cert['certonly']) {
        create_resources(::letsencrypt::certonly, $cert['certonly'])
      }
    }
  }
}
```

The line within the each loop ```create_resources(::letsencrypt::certonly, $cert['certonly'])```
sends our hiera to the letsencrypt module to create the certificate settings which is the equivalent of the following puppet code:

```ruby
letsencrypt::certonly { 'test.andrewrowe.dev':
  ensure               => present,
  domains              => ['test.andrewrowe.dev'],
  plugin               => 'webroot',
  webroot_paths        => ['/var/www/letsencrypt/'],
  manage_cron          => true,
  cron_hour            => 0,
  suppress_cron_output => true,
  deploy_hook_commands => ['/bin/systemctl reload nginx.service']
}

letsencrypt::certonly { 'test1.andrewrowe.dev':
  ensure               => present,
  domains              => ['test1.andrewrowe.dev'],
  plugin               => 'webroot',
  webroot_paths        => ['/var/www/letsencrypt/'],
  manage_cron          => true,
  cron_hour            => 0,
  suppress_cron_output => true,
  deploy_hook_commands => ['/bin/systemctl reload nginx.service']
}
```
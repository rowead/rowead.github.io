---
layout: article
title: Puppet and Letsencrypt
excerpt: >
  Simple(ish) way to get letsencrypt certs and set up auto renewal.
published: true
---
A work in progress...

```yaml
---
classes:
  - letsencrypt
  - nginx
  - andrewr::letsencrypt

letsencrypt::email: '------@gmail.com'
letsencrypt::package_ensure: latest

andrewr::letsencrypt:
  certs:
    'test.andrewrowe.dev':
      certonly:
        'test.andrewrowe.dev':
          ensure: present
          domains:
            - 'test.andrewrowe.dev'
          manage_cron: true
          cron_hour: 0
          suppress_cron_output: true
          pre_hook_commands:
            - 'service nginx stop'
          post_hook_commands:
            - 'service nginx start'
    'test1.andrewrowe.dev':
      certonly:
        'test1.andrewrowe.dev':
          ensure: present
          domains:
            - 'test1.andrewrowe.dev'
          manage_cron: true
          cron_hour: 0
          suppress_cron_output: true
          pre_hook_commands:
            - 'service nginx stop'
          post_hook_commands:
            - 'service nginx start'

nginx::ssl_protocols: 'TLSv1.2 TLSv1.3'
nginx::ssl_ciphers: 'TLS13-CHACHA20-POLY1305-SHA256:TLS13-AES-256-GCM-SHA384:TLS13-AES-128-GCM-SHA256:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA:ECDHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA256:DHE-RSA-AES256-SHA:ECDHE-ECDSA-DES-CBC3-SHA:ECDHE-RSA-DES-CBC3-SHA:EDH-RSA-DES-CBC3-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:DES-CBC3-SHA:!DSS'


nginx::nginx_servers:
  'test.andrewrowe.dev':
    ensure: present
    ssl: true
    listen_port: 80
    ssl_port: 443
    ssl_redirect: true
    use_default_location: false
    www_root: '/var/www/test.andrewrowe.dev'
    ssl_cert: "/etc/letsencrypt/live/test.andrewrowe.dev/fullchain.pem"
    ssl_key: "/etc/letsencrypt/live/test.andrewrowe.dev/privkey.pem"
  'test1.andrewrowe.dev':
    ensure: present
    ssl: true
    listen_port: 80
    ssl_port: 443
    ssl_redirect: true
    use_default_location: false
    www_root: '/var/www/test.andrewrowe.dev'
    ssl_cert: "/etc/letsencrypt/live/test1.andrewrowe.dev/fullchain.pem"
    ssl_key: "/etc/letsencrypt/live/test1.andrewrowe.dev/privkey.pem"
```

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

  if ( $letsencrypt['certs'] ) {
    each($letsencrypt['certs']) |$key, $cert| {
      if ($cert['certonly']) {
        create_resources(::letsencrypt::certonly, $cert['certonly'])
      }
    }
  }
}
```

Check out the results here: [test.andrewrowe.dev](https://test.andrewrowe.dev)
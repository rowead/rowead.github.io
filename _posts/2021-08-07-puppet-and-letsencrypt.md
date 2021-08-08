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

andrewr::letsencrypt:certonly:
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


andrewr::ssl_cert: '/etc/letsencrypt/live/test.andrewrowe.dev/fullchain.pem'
andrewr::ssl_key: '/etc/letsencrypt/live/test.andrewrowe.dev/privkey.pem'

nginx::nginx_servers:
  'test.andrewrowe.dev':
    ensure: present
    ssl: true
    listen_port: 80
    ssl_port: 443
    ssl_redirect: true
    use_default_location: false
    www_root: '/var/www/test.andrewrowe.dev'
    ssl_cert: "%{alias('andrewr::ssl_cert')}"
    ssl_key: "%{alias('andrewr::ssl_key')}"
```

```ruby
# @summary A short summary of the purpose of this class
#
# A description of what this class does
#
# @example
#   include andrewr::letsencrypt
class andrewr::letsencrypt {
  $certs = lookup({
    'name'  => 'andrewr::letsencrypt:certonly',
    'merge' => {
      'strategy' => 'deep',
    },
    default_value => {}
  })

  if ( $certs ) {
    create_resources(::letsencrypt::certonly, $certs)
  }
}
```

Check out the results here: [test.andrewrowe.dev](https://test.andrewrowe.dev)
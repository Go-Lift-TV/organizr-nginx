# organizr-nginx

This repo contains the custom Go Lift TV Nginx configs that reverse proxy and run
a few dozen apps on an unRAID server with the lsio/swag and Organizr Docker
containers.

At some point a better HOWTO, video, or blog post will be created to explain how
to put all these pieces together. For now, here's the configs. Enjoy!

I didn't use any of the ngnix configs that come with the letsencrypt container.
You can just copy these into appdata/letsencrypt/nginx to get going.

[NextCloud](http://nextcloud.com), [Chevereto](http://chevereto.com),
[DokuWiki](http://dokuwiki.org),   and [YOURLS](https://yourls.org)
are installed directly into the letsencrypt config folder, under `www/`
at `nc/`, `img/`, `wiki/`, and `YOURLS/` respectively.

The rest of the apps, including [Organizr](https://organizr.app) are running in
their own Docker containers, or in the case of [SecuritySpy](http://bensoftware.com/securityspy/)
and [Indigo](http://indigodomo.com), different servers (on the same network).

All of the containers are running in a custom bridge so DNS is working and this
is how nginx finds all the backends. Take note of [proxy.conf](proxy.conf).

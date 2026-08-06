# organizr-nginx

Custom Go Lift TV nginx configs that reverse proxy a few dozen apps on an
unRAID server using the lsio/swag and Organizr Docker containers.

These live in `appdata/swag/nginx` inside the SWAG container (`/config/nginx`).

## DNS triad

Every service name resolves three ways; all of them terminate on the same SWAG
nginx with a cert that covers each name (`SUBDOMAINS=*.home,*.slvr,...`):

| Name | Path | Notes |
|------|------|-------|
| `app.golift.tv` | Cloudflare → router :443 → SWAG | Public. Cloudflare IPs are trusted for `real_ip` (see `site-confs/00-custom.conf`). |
| `app.home.golift.tv` | Router (direct, no Cloudflare) → SWAG | Used by mobile apps (SecuritySpy, Home Assistant). |
| `app.slvr.golift.tv` | Private IP → SWAG | LAN only. |

## Authentication

Organizr fronts most apps via nginx `auth_request` (`auth/*.conf`):

| Include | Organizr group | Used by |
|---------|----------------|---------|
| `auth/admin.conf` | 0 Admin | UniFi, Notifiarr |
| `auth/coadmin.conf` | 1 Co-Admin | — |
| `auth/superuser.conf` | 2 Super User | Sonarr/Radarr/Lidarr/Readarr, Jackett, Prowlarr, Deluge, NZBGet |
| `auth/poweruser.conf` | 3 Power User | Netdata, AWStats, Bazarr |
| `auth/user.conf` | 4 User | SecuritySpy, YOURLS admin |
| `auth/wideopen.conf` | 999 | Guests |

SecuritySpy gets per-group credentials injected: the `$group →
$secspybase64_password` map in `site-confs/00-custom.conf` picks an `org_*`
SecuritySpy user out of `golift/secrets.conf` (git-ignored, server only).

Apps with their own auth and no Organizr wall: SecuritySpy on `ss.*` (mobile
apps), Home Assistant on `ha.home.*`/`ha.slvr.*`, Gitea (public), Plex, YOURLS
redirects on `l.*`.

`ha.golift.tv` (public) is LAN-locked until the Alexa/mTLS project lands; see
`site-confs/homeassistant.conf` and
[TwitchCaptain/alexa-homeassistant](https://github.com/TwitchCaptain/alexa-homeassistant).

## Change control

The live server copy is a git checkout. Never edit prod directly:

1. Open a PR against this repo and review it.
2. After merge: `ssh slvr`, then `cd /mnt/cache/appdata/swag/nginx && sudo -u nobody git pull`.
3. Test and reload: `docker exec swag nginx -t && docker exec swag nginx -s reload`.

## Layout

| Path | Purpose |
|------|---------|
| `nginx.conf`, `ssl.conf`, `proxy.conf`, `resolver.conf` | Core config. Upstream backends are `set` variables in `proxy.conf`. |
| `site-confs/` | One file per vhost (`default.conf` is the main Organizr domain). |
| `golift/` | Per-app snippets included into the main domain's server block. |
| `auth/` | Organizr `auth_request` includes, one per group level. |
| `golift/secrets.conf` | **Not in git.** `set` variables holding base64 basic-auth credentials. |

[NextCloud is gone.] [Chevereto](http://chevereto.com), [DokuWiki](http://dokuwiki.org),
and [YOURLS](https://yourls.org) are installed directly in the SWAG container
under `www/` at `img/`, `wiki/`, and `yourls/` respectively. Everything else
runs in its own container (custom bridge network, so Docker DNS resolves the
`proxy.conf` names), except SecuritySpy and Home Assistant which run on other
hosts on the LAN.

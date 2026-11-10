# Configuration

`update-zonefile.py` reads `config.yml` next to the script, or the file given
with `--config`. Each config builds one zone. Relative `file:` and `cache`
paths are relative to the script's directory.

| Key | Default | Meaning |
|---|---|---|
| `req_timeout_s` | `10` | HTTP timeout per download, seconds |
| `wildcard_block` | `False` | also block `*.name`, i.e. everything under a listed name; names under another listed name are then dropped as redundant |
| `cache` | `.cache/bind_rpz_builder` | downloaded lists (the last good copy) and temporary zone files |
| `blocking_mode` | `NXDOMAIN` | `NXDOMAIN` answers `CNAME .`; any other value sinkholes to `A 0.0.0.0` / `AAAA ::` |
| `lists` | `[]` | the blocklists, below |
| `exclude_lists` | `[]` | names another zone already blocks, [below](#exclude-lists) |
| `domain_whitelist` | `[]` | names never blocked (exact match) |

## Lists

Each entry has `url:` or `file:`, and optionally:

- `format:`
  - `domain`: a name per line (default)
  - `hosts`: hosts file, `0.0.0.0 name`
  - `infoblox`: Infoblox CSV, the name in the second column
- `required: true`: if the list is missing or empty, leave the zone as it
  is and exit 1. Use it for local files another job writes, so a failed
  job doesn't empty the zone.

`#` comments and inline comments are ignored. Downloads send
`If-Modified-Since`; a failed download uses the cached copy. Names with
`_` or that fail validation are skipped and reported.

Names are compared lower-case, and a name in several lists is written once.

## Exclude lists

`exclude_lists` takes entries like `lists` (`url:` or `file:`, `format:`
`domain` or `hosts`). A name they list, and everything under it, is left
out of this zone: they are meant to be another zone's lists, blocked there
with `wildcard_block`, that BIND checks first. The ad zone uses it to leave
out what the [threat zone](threat-zone.md) blocks:

```yaml
exclude_lists:
  - file: '/var/lib/threat-multimap/malware.map'
  - file: '/var/lib/threat-multimap/phishing.map'
  - file: '/var/lib/threat-multimap/fraud.map'
```

A missing exclude list is reported and skipped. The lists are read a line
at a time and only names that match this zone are kept, so millions of
excluded names cost little memory. The exclusion is as of this zone's
build: a name the threat feeds drop goes back to being blocked as an ad
at the ad zone's next build.

## Default ad and tracking lists (`config.yml`)

- `blocklist.txt` (local)
- [Peter Lowe's ad and tracking server list](https://pgl.yoyo.org/adservers/)
- [MVPS HOSTS](http://winhelp2002.mvps.org/)
- [AdAway](https://adaway.org/hosts.txt)
- [Dan Pollock's hosts file](https://www.someonewhocares.org/hosts/zero/)
- [StevenBlack unified hosts](https://github.com/StevenBlack/hosts)
- [Disconnect.me tracking and ads](https://disconnect.me/trackerprotection)
- [HaGeZi Ultimate](https://github.com/hagezi/dns-blocklists)
- [The Block List Project ads](https://blocklistproject.github.io/Lists/)

Phishing, malware and scam lists are in the [threat zone](threat-zone.md).

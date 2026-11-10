# Configuration

`update-zonefile.py` reads `config.yml` next to the script, or the file given
with `--config`. Each config builds one zone. Relative `file:` and `cache`
paths are relative to the script's directory.

| Key | Default | Meaning |
|---|---|---|
| `req_timeout_s` | `10` | HTTP timeout per download, seconds |
| `wildcard_block` | `False` | also block `*.name`, i.e. everything under a listed name |
| `cache` | `.cache/bind_rpz_builder` | downloaded lists (the last good copy) and temporary zone files |
| `blocking_mode` | `NXDOMAIN` | `NXDOMAIN` answers `CNAME .`; any other value sinkholes to `A 0.0.0.0` / `AAAA ::` |
| `lists` | `[]` | the blocklists, below |
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

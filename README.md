# bind-rpz-builder

Builds BIND [response policy zones](https://en.wikipedia.org/wiki/Response_policy_zone)
(RPZ) from public blocklists. Each zone has its own config file:

| Zone | Config | Blocks |
|---|---|---|
| `rpz.adblock` | [`config.yml`](config.yml) | ads and tracking |
| `rpz.threat` | [`config-threat.yml`](config-threat.yml) | phishing, malware, scams, from [rspamd-threat-multimap](https://github.com/nabbi/rspamd-threat-multimap) |

A listed name gets `NXDOMAIN` (or `0.0.0.0` / `::`), optionally along with
everything under it. The zone is checked with `named-checkzone` and loaded
with `rndc reload`. Requires BIND 9.8+ and Python 3.

## Quick start

```bash
python3 -m venv .venv && .venv/bin/pip install -r requirements.txt

.venv/bin/python update-zonefile.py /etc/bind/db.rpz.adblock rpz.adblock
.venv/bin/python update-zonefile.py --config config-threat.yml /etc/bind/db.rpz.threat rpz.threat
```

The first run creates the zone file and prints the `named.conf` lines to add.

## Docs

- [Configuration](docs/configuration.md): config keys, list formats, the default ad lists
- [Usage](docs/usage.md): options, scheduling
- [BIND](docs/bind.md): `named.conf`, zone order, views, allowlisting
- [Threat zone](docs/threat-zone.md): building `rpz.threat` from rspamd-threat-multimap
- [Docker](docs/docker.md)

## License

MIT, see [LICENSE](LICENSE). Forked from
[Trellmor/bind-adblock](https://github.com/Trellmor/bind-adblock)
(Copyright (c) 2018 Daniel Triendl).

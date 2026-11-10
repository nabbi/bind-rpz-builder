# Usage

```text
usage: update-zonefile.py [-h] [--no-bind] [--raw] [--empty] [--views VIEWS]
                          [-c CONFIG]
                          zonefile origin

positional arguments:
  zonefile    path to zone file
  origin      zone origin

options:
  --no-bind   Don't check the zone or reload it in BIND
  --raw       Save the zone file in raw format (needs named-compilezone)
  --empty     Write a header-only (empty) zone
  --views     BIND views the zone is defined in, space separated
  -c CONFIG, --config CONFIG
              config file (default: config.yml next to this script)
```

A run:

1. downloads and parses the lists, keeping only unique valid names;
2. writes the zone with its serial incremented;
3. checks it with `named-checkzone` (if the check fails, the old zone stays);
4. runs `restorecon` on it if SELinux is enforcing;
5. runs `rndc reload <origin>`, once per view with `--views`.

`--no-bind` skips steps 3 to 5, e.g. for a first build before `named.conf`
has the zone. Builds of different zones can share a cache directory.

## Scheduling

Run each zone from cron, daily for ads and every few hours for threats. See
[`crontab/`](../crontab):

```bash
5 2 * * * cd /opt/bind-rpz-builder && ./venv/bin/python3 update-zonefile.py /var/bind/pri/rpz-adblock.zone rpz.adblock > /var/log/rpz-adblock.log 2>&1
```

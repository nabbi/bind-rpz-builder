# Threat zone

`config-threat.yml` builds `rpz.threat` from the maps written by
[rspamd-threat-multimap](https://github.com/nabbi/rspamd-threat-multimap).
It writes one map per category (`malware.map`, `phishing.map`,
`fraud.map`), each a domain per line, from:

- phishing feeds: OpenPhish/PhishTank (malware-filter), CERT Polska
- malware feeds: URLhaus, Infoblox, The Block List Project ransomware
- vendor IOC repositories: Cisco Talos, Unit 42, ESET, Bitdefender, Sophos,
  Zscaler, Netskope
- optionally, fake shop and scam lists (HaGeZi, The Block List Project)

It leaves out the shared hosts that IOC lists name as a payload URL
(`drive.google.com`, `github.com`, `t.me`, a tenant's `workers.dev`, and
about 270 more). Blocking one of those outright would break the service
for everyone. It also has its own allowlist, which covers each listed
name and everything under it.

## Setup

1. Install rspamd-threat-multimap and point `output_dir` in its
   `config.toml` at the directory `config-threat.yml` reads
   (`/var/lib/threat-multimap`). No rspamd is needed: the `symbol` of each
   category is required by the config but unused.
2. Enable the fraud feeds if you want them (they are off in its example).
3. Refresh the maps, then build the zone:

```bash
rspamd_threat_multimap.py -c /etc/threat-multimap/config.toml update
update-zonefile.py --config config-threat.yml /etc/bind/db.rpz.threat rpz.threat
```

`update` exits 1 when a feed failed. It then uses that feed's last good
copy, so build the zone anyway. Its maps are rewritten only when their
names change, so a scheduled job can rebuild the zone only when a `.map`
is newer than the zone file.

The maps are `required: true` in `config-threat.yml`: if one is missing,
the zone is not replaced.

## Size

Measured 2026-11-09:

| Feeds | Names | Map update peak RAM | Zone records |
|---|---:|---:|---:|
| multimap example, plus the fraud feeds | ~237,000 | ~70 MB | ~475,000 |
| the same, plus The Block List Project malware list | ~2,800,000 | ~630 MB | ~5,600,000 |

With `wildcard_block`, every name has two records, so the zone holds
twice as many records as there are names.

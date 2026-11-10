# BIND

## Zones

Name the zones in `response-policy`. The first zone that matches wins, so
put local overrides first, then threats, then ads. That way a name on
both lists is logged as a threat:

```bash
options {
    response-policy {
        zone "rpz.local";      // your own passthru/override rules
        zone "rpz.threat";
        zone "rpz.adblock";
    };
};

zone "rpz.threat" {
    type master;
    file "/etc/bind/db.rpz.threat";
    masterfile-format text;
    allow-query { none; };
};

zone "rpz.adblock" {
    type master;
    file "/etc/bind/db.rpz.adblock";
    masterfile-format text;
    allow-query { none; };
};
```

The first run of `update-zonefile.py` creates a missing zone file. To
create one by hand:

```text
@ 3600 IN SOA @ admin.example.com. 0 86400 7200 2592000 86400
@ 3600 IN NS LOCALHOST.
```

To log hits, use the `rpz` category:

```bash
logging {
    channel rpz { file "/var/log/bind/rpz.log" versions 5 size 50m; print-time yes; };
    category rpz { rpz; };
};
```

## Views

If a zone is defined in more than one view, pass the views so each one
reloads:

```bash
update-zonefile.py --views "internal dmz" /etc/bind/db.rpz.adblock rpz.adblock
```

Without `--views`, `rndc` fails with `zone 'rpz.adblock' was found in multiple views`.

## Allowlisting

- `domain_whitelist` in the zone's config (exact names), or
- a `rpz-passthru.` rule in a zone listed first, e.g. `rpz.local`:

```text
example.com     IN CNAME rpz-passthru.
*.example.com   IN CNAME rpz-passthru.
```

For the threat zone, use rspamd-threat-multimap's allowlist instead: it
also drops the names under a listed one.

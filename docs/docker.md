# Docker

[`docker/Dockerfile`](../docker/Dockerfile) and
[`docker/docker-compose.yml`](../docker/docker-compose.yml) are a starting
point for running the builder in a container; expect to adapt them for
production. The image builds the ad zone (`config.yml`) once, with
`--no-bind`, into `/zones/rpz-adblock.zone`, then exits. `/zones` is either a
named volume shared with a bind9 container or a host path such as `/var/bind`.

```shell
cd docker
docker-compose build
docker-compose up updater          # build the zone once
```

The image has no scheduler and doesn't reload BIND. Run it from the host's
cron, then restart or reload bind9:

```shell
docker-compose up updater >/dev/null 2>&1 && docker-compose restart bind9 >/dev/null 2>&1
```

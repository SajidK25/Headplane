# Headplane

- Goto Coolify Dashboard and Click on the Terminal tab from left sidebar menu
- Select localhost from the dropdwon
- when terminal open then execute `docker ps` to list the all running containers
```
root@jenkin-master:~# docker ps
CONTAINER ID   IMAGE                                        COMMAND                  CREATED        STATUS                  PORTS                                                                                                                                                                NAMES
67a41cc80ec3   uwo88og4o8o0kks0gcw84k84-headplane           "/nodejs/bin/node /a…"   3 hours ago    Up 3 hours (healthy)    3000/tcp                                                                                                                                                             headplane-uwo88og4o8o0kks0gcw84k84-130548131589
4f3c692ff544   uwo88og4o8o0kks0gcw84k84-headscale           "/ko-app/headscale s…"   3 hours ago    Up 3 hours (healthy)    8080/tcp                                                                                                                                                             headscale-uwo88og4o8o0kks0gcw84k84-130548125347
3d9819ba1a8e   traefik:v3.6                                 "/entrypoint.sh --pi…"   21 hours ago   Up 21 hours (healthy)   0.0.0.0:80->80/tcp, [::]:80->80/tcp, 0.0.0.0:443->443/tcp, [::]:443->443/tcp, 0.0.0.0:8080->8080/tcp, [::]:8080->8080/tcp, 0.0.0.0:443->443/udp, [::]:443->443/udp   coolify-proxy
5daa0cff4f78   cloudflare/cloudflared:latest                "cloudflared --no-au…"   22 hours ago   Up 22 hours (healthy)                                                                                                                                                                        cloudflared-h80sowo00wwcsowggow44ck0
f1a6129ead41   ghcr.io/coollabsio/sentinel:0.0.18           "/app/sentinel"          2 days ago     Up 16 hours (healthy)                                                                                                                                                                        coolify-sentinel
59fc66ab812f   ghcr.io/coollabsio/coolify:4.0.0-beta.452    "docker-php-serversi…"   2 days ago     Up 2 days (healthy)     8000/tcp, 8443/tcp, 9000/tcp, 0.0.0.0:8000->8080/tcp, [::]:8000->8080/tcp                                                                                            coolify
13820cc4c4a3   postgres:15-alpine                           "docker-entrypoint.s…"   2 days ago     Up 2 days (healthy)     5432/tcp                                                                                                                                                             coolify-db
66f2358a6dc9   redis:7-alpine                               "docker-entrypoint.s…"   2 days ago     Up 2 days (healthy)     6379/tcp                                                                                                                                                             coolify-redis
4189b921dbce   ghcr.io/coollabsio/coolify-realtime:1.0.10   "/bin/sh /soketi-ent…"   2 days ago     Up 2 days (healthy)     0.0.0.0:6001-6002->6001-6002/tcp, [::]:6001-6002->6001-6002/tcp                                                                                                      coolify-realtime
```
- copy the name of Headscale container `headscale-uwo88og4o8o0kks0gcw84k84-130548125347` 
- Execute this command to get Headscale apikey `docker exec -it headscale-uwo88og4o8o0kks0gcw84k84-130548125347 headscale apikey create`
```
# Headscale apikey 
t3Xrw-z.3M1kevwRK0K06bab_65OmxhxaO2OqHbM
```
- Copy this apikey and use it as Headplane login password

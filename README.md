# Docker Compose
It is important that docker-compose.yml for haproxy has this otherwise it wont detect the
echo servers.
```
    depends_on:
     - echo
```
# Static
haproxy.cfg
```
global
    log          fd@2 local2
    stats timeout 2m
    spread-checks 15

defaults
    log global
    mode http
    option httplog
    timeout connect 5s
    timeout check 5s
    timeout client 2m
    timeout server 2m

listen stats
    bind :1936
    mode http
    stats enable
    stats hide-version
    stats realm Haproxy\ Statistics
    stats uri /

frontend default
    bind *:80
    default_backend echo

backend echo
    balance roundrobin
    option httpchk GET /
    server echo1 haproxy-dynamic-echo-1:80 check inter 30s
    server echo2 haproxy-dynamic-echo-2:80 check inter 30s
```

# Dynamic
```
resolvers docker
    nameserver dns1 127.0.0.11:53
    timeout retry   1s
    resolve_retries 10
    hold valid 1s

global
    log          fd@2 local2
    stats timeout 2m
    spread-checks 15

defaults
    log global
    mode http
    option httplog
    timeout connect 5s
    timeout check 5s
    timeout client 2m
    timeout server 2m
    default-server init-addr none

listen stats
    bind :1936
    mode http
    stats enable
    stats hide-version
    stats realm Haproxy\ Statistics
    stats uri /

frontend default
    bind *:80
    default_backend echo

backend echo
    balance roundrobin
    mode http
    option httpchk GET /
    server-template echo 5 echo:80 check inter 10s resolvers docker resolve-prefer ipv4
```

# How to test
- `curl localhost`
- navigate to `localhost:1937` to see Stats (stats/stats).
- scale echo server `docker-compose up -d --scale echo=5` then refresh `localhost:1937`.
- use load test tool (wrk, jmeter) to make requests to `localhost`.
- run `docker-compose logs -f` to see the roundrobin.
# INFLUXDB
Matrix or Data Base

######################## Crear Carpetas #######################

```
mkdir -p /docker-data/influx1/etc
mkdir -p /docker-data/influx1/backup				
mkdir -p /data/influx1
```

####################### Configurar influxdb.conf ###################

```
nano /docker-data/influx1/etc/influxdb.conf
```

---------------------- Archivo influxdb.conf --------------------

```
reporting-disabled = false
bind-address = "127.0.0.1:8088"
[meta]
dir = "/var/lib/influxdb/meta"
  retention-autocreate = true
  logging-enabled = true
[data]
dir = "/var/lib/influxdb/data"
  index-version = "inmem"
  wal-dir = "/var/lib/influxdb/wal"
  wal-fsync-delay = "0s"
  query-log-enabled = true
  cache-max-memory-size = 1073741824
  cache-snapshot-memory-size = 26214400
  cache-snapshot-write-cold-duration = "10m0s"
  compact-full-write-cold-duration = "4h0m0s"
  max-series-per-database = 1000000
  max-values-per-tag = 100000
  max-concurrent-compactions = 0
  trace-logging-enabled = false
[coordinator]
  write-timeout = "10s"
  max-concurrent-queries = 0
  query-timeout = "0s"
  log-queries-after = "0s"
  max-select-point = 0
  max-select-series = 0
  max-select-buckets = 0
[retention]
  enabled = true
  check-interval = "30m0s"
[shard-precreation]
  enabled = true
  check-interval = "10m0s"
  advance-period = "30m0s"
[monitor]
  store-enabled = true
  store-database = "_internal"
  store-interval = "10s"
[subscriber]
  enabled = true
  http-timeout = "30s"
  insecure-skip-verify = false
  ca-certs = ""
  write-concurrency = 40
  write-buffer-size = 1000
[http]
  enabled = true
  bind-address = ":8086"
  auth-enabled = true
  log-enabled = true
  write-tracing = false
  pprof-enabled = true
  https-enabled = false
  https-certificate = "/etc/ssl/influxdb.pem"
  https-private-key = ""
  max-row-limit = 0
  max-connection-limit = 0
  shared-secret = ""
  realm = "InfluxDB"
  unix-socket-enabled = false
  bind-socket = "/var/run/influxdb.sock"
[[graphite]]
  enabled = false
  bind-address = ":2003"
  database = "graphite"
  retention-policy = ""
  protocol = "tcp"
  batch-size = 5000
  batch-pending = 10
  batch-timeout = "1s"
  consistency-level = "one"
  separator = "."
  udp-read-buffer = 0
[[collectd]]
  enabled = false
  bind-address = ":25826"
  database = "collectd"
  retention-policy = ""
  batch-size = 5000
  batch-pending = 10
  batch-timeout = "10s"
  read-buffer = 0
  typesdb = "/usr/share/collectd/types.db"
  security-level = "none"
  auth-file = "/etc/collectd/auth_file"
[[opentsdb]]
  enabled = false
  bind-address = ":4242"
  database = "opentsdb"
  retention-policy = ""
  consistency-level = "one"
  tls-enabled = false
  certificate = "/etc/ssl/influxdb.pem"
  batch-size = 1000
  batch-pending = 5
  batch-timeout = "1s"
  log-point-errors = true
[[udp]]
  enabled = false
  bind-address = ":8089"
  database = "udp"
  retention-policy = ""
  batch-size = 5000
  batch-pending = 10
  read-buffer = 0
  batch-timeout = "1s"
  precision = ""
[continuous_queries]
  log-enabled = true
  enabled = true
  run-interval = "1s"
```

####################### Crear Red de Docker ###################

```
docker network ls
docker network create i40sys
```

####################### Crear Docker-compose ##################

```
nano /docker-data/influx1/docker-compose.yml
```

---------------------- Archivo docker-compose.yml ----------------

```
version: '3.8'
services:
 influx1:
   container_name: influx1
   image: influxdb:1.8
   restart: unless-stopped
   environment: 
      - TZ="America/Santiago"
   volumes:
      - /data/influx1:/var/lib/influxdb
      - ./etc/influxdb.conf:/etc/influxdb/influxdb.conf:ro
      - ./backup:/backup
      - /etc/localtime:/etc/localtime:ro
   command: -config /etc/influxdb/influxdb.conf
   networks:
      - i40sys
networks:
 i40sys:
   external:
     name: i40sys
```

#################### ejecutamos docker-compose ##############

```
cd /docker-data/influx1/

docker-compose up 
```

#################### En otra terminal #######################

```
docker exec -ti influx1 /usr/bin/influx

create user "admin" with password 'password' with all privileges;
```

#################### ejecutar docker ##########################

```
docker restart influx1
docker exec -ti influx1 /usr/bin/influx
```

-------------------- crear base de datos -----------------------

```
  auth
  admin
  password
  
CREATE DATABASE "telegraf" WITH DURATION 4w1d REPLICATION 1 NAME "one_month"
  show databases
  use telegraf
  
  SHOW RETENTION POLICIES
  
  create user "telegraf_r" with password 'password';
  create user "telegraf_w" with password 'password';
  
  grant read on "telegraf" to "telegraf_r";
  grant write on "telegraf" to "telegraf_w";

$ show users
```

# sensu server installation - el7
## system related
```
systemctl stop firewalld
systemctl disable firewalld
yum install -y epel-release
yum makecache fast
yum install -y vim wget curl erlang socat openssl
echo "127.0.0.1 redis sensu api" >> /etc/hosts
```

## redis
```
yum install -y redis
cat << EOF > /etc/redis.conf
protected-mode yes
port 6379
tcp-backlog 511
timeout 0
tcp-keepalive 300
daemonize no
supervised no
pidfile /var/run/redis/redis.pid
loglevel notice
logfile /var/log/redis/redis.log
databases 16
save 900 1
save 300 10
save 60 10000
stop-writes-on-bgsave-error yes
rdbcompression yes
rdbchecksum yes
dbfilename dump.rdb
dir /var/lib/redis
slave-serve-stale-data yes
slave-read-only yes
repl-diskless-sync no
repl-diskless-sync-delay 5
repl-disable-tcp-nodelay no
slave-priority 100
appendonly no
appendfilename "appendonly.aof"
appendfsync everysec
no-appendfsync-on-rewrite no
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
aof-load-truncated yes
lua-time-limit 5000
slowlog-log-slower-than 10000
slowlog-max-len 128
latency-monitor-threshold 0
notify-keyspace-events ""
hash-max-ziplist-entries 512
hash-max-ziplist-value 64
list-max-ziplist-size -2
list-compress-depth 0
set-max-intset-entries 512
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
hll-sparse-max-bytes 3000
activerehashing yes
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit slave 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60
hz 10
aof-rewrite-incremental-fsync yes
EOF
systemctl start redis
```

## sensu server
```
rpm --import http://repos.sensuapp.org/apt/pubkey.gpg
echo '[sensu]
name=sensu
baseurl=https://sensu.global.ssl.fastly.net/yum/$releasever/$basearch/
gpgcheck=0
enabled=1' | sudo tee /etc/yum.repos.d/sensu.repo
yum makecache fast
yum install -y sensu uchiwa
/opt/sensu/embedded/bin/gem install --no-ri --no-rdoc sensu-plugins-memory-checks
/opt/sensu/embedded/bin/gem install --no-ri --no-rdoc sensu-plugins-disk-checks
/opt/sensu/embedded/bin/gem install --no-ri --no-rdoc sensu-plugins-cpu-checks
/opt/sensu/embedded/bin/gem install --no-ri --no-rdoc sensu-plugins-load-checks
/opt/sensu/embedded/bin/gem install --no-ri --no-rdoc sensu-plugins-slack
```

# server configuration
## redis
```
cat << EOF > /etc/sensu/conf.d/redis.json
{
    "redis": {
        "host": "redis",
        "port": 6379,
        "db": 0,
        "auto_reconnect": true,
        "reconnect_on_error": false
    }
}
EOF
cat << EOF > /etc/sensu/conf.d/transport.json
{
    "transport": {
      "name": "redis",
      "reconnect_on_error": true
    }
}
EOF
```
h2. sensu 
```
cat << EOF > /etc/sensu/conf.d/api.json
{
    "api": {
        "host": "api",
        "port": 4567
    }
}
EOF
cat << EOF > /etc/sensu/conf.d/checks.json

{
  "checks": {
    "Memory_Percent": {
      "command": "/opt/sensu/embedded/bin/check-memory-percent.rb -w 75 -c 90",
      "interval": 30,
      "occurrences": 2,
      "type": "standard",
      "handlers": [ "slack" ],
      "subscribers": [ "all" ]
    },
    "CPU": {
      "command": "/opt/sensu/embedded/bin/check-cpu.rb -w 85 -c 90",
      "interval": 30,
      "occurrences": 2,
      "type": "standard",
      "handlers": [ "slack" ],
      "subscribers": [ "all" ]
    },
    "Load": {
      "command": "/opt/sensu/embedded/bin/check-load.rb",
      "interval": 30,
      "occurrences": 2,
      "type": "standard",
      "handlers": [ "slack" ],
      "subscribers": [ "all" ]
    },
    "Disk_Usage": {
      "command": "/opt/sensu/embedded/bin/check-disk-usage.rb -w 90 -c 95",
      "interval": 30,
      "occurrences": 2,
      "handlers": [ "slack" ],
      "subscribers": [ "all" ]
    }
  }
}
EOF
cat << EOF > /etc/sensu/conf.d/handler-slack.json
{
  "handlers": {
    "slack": {
      "type": "pipe",
      "command": "/opt/sensu/embedded/bin/handler-slack-multichannel.rb",
      "severities": [ "critical" ]
      }
    },
  "slack": {
    "channels": {
      "default": [ "#sensu" ],
      "production": [ "#sensu" ],
      "testing": [ "#sensu" ]
    },
    "message_prefix": "@channel",
    "webhook_url": "https://hooks.slack.com/services/sometoken"
  }
}
EOF
```

## uchiwa
```
cat << EOF > /etc/sensu/uchiwa.json
{

  "sensu": [
    {
      "name": "test",
      "host": "api",
      "port": 4567
    }
  ],

  "uchiwa": {
    "host": "0.0.0.0",
    "port": 3000,
    "refresh": 10,
    "loglevel": "info"
  }
}
EOF
```

# client configuration
## sensu server as a client
```
cat << EOF > /etc/sensu/conf.d/client.json
{
  "client": {
    "name": "sensuserver",
    "address": "localhost",
    "subscriptions": [ "ALL" ]
  }
}
EOF
```

## starting and enabling
```
# server
systemctl start redis
systemctl start sensu-server
systemctl start sensu-api
systemctl start uchiwa

systemctl enable redis
systemctl enable sensu-server
systemctl enable sensu-api
systemctl enable uchiwa

systemctl stop redis
systemctl stop sensu-server
systemctl stop sensu-api
systemctl stop uchiwa

# client
systemctl stop sensu-client
systemctl start sensu-client
systemctl enable sensu-client
```

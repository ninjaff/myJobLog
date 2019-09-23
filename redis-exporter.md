---
layout: service
title: redis-exporter
date: 2019-08-23 08:16:37
tags: [prometheus, exporter, redis]
---

### install golang source
```bash
$ go get github.com/oliver006/redis_exporter
$ cd $GOPATH/src/github.com/oliver006/redis_exporter
$ go build
$ ./redis_exporter <flags>

## 无密码
$ ./redis_exporter -redis.addr 127.0.0.1:6379 &
## 有密码
$ redis_exporter -redis.addr 127.0.0.1:6379 -redis.password 123456 
## 后台启动 redis.addr defaults to `redis://localhost:6379`
nohup ./redis_exporter -redis.password 123456
```

### add prometheus redis config
```yaml
scrape_configs:
...
- job_name: redis_exporter
  static_configs:
  - targets: ['127.0.0.1:9121']
    labels:
      instance: redis1
...
```

### download grafana dashbord config import
https://grafana.com/dashboards/763

wget https://grafana.com/api/dashboards/763/revisions/1/download

### 
https://github.com/prometheus/docs/blob/master/content/docs/instrumenting/exporters.md

https://github.com/oliver006/redis_exporter

---
title: prometheus-alertManager
date: 2019-08-28 08:20:00
tags: [prometheus, alertManager]
---
### install 

tar xf alertmanager-x.x.x.linux-amd64.tar.gz

### configer rules
vi /etc/prometheus/alert.rules
```
groups:
  - name: web.hook
    rules:
    # Alert for any instance that is unreachable for >1 minutes.
    - alert: InstanceDown
      expr: up == 0
      for: 1m
      labels:
        severity: page
      annotations:
        summary: "Instance {{ $labels.instance }} down"
        description: "{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 5 minutes."
```

### configer prometheus.yml
prometheus.yml
```
...
alerting:
  alertmanagers:
  - scheme: http
    static_configs:
    - targets:
      - "localhost:9093"
rule_files:
  - /etc/prometheus/alert.rules
...
```

### configer alertmanager.yml
```
global:
  smtp_smarthost: 'smtp.qq.com:587'
  smtp_from: 'xxx@qq.com'
  smtp_auth_username: 'xxx@qq.com'
  smtp_auth_password: 'password'//该密码不是邮箱密码，是邮箱授权码
  resolve_timeout: 5m
route:
  group_by: ['alertname']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 1h
  receiver: 'web.hook'
receivers:
  - name: 'web.hook'
    email_configs:
    - to: 'example@qq.com'//接收预警邮件的邮箱
inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'dev', 'instance']
```

### test
通过kill被监控端的node_exporter进程可以试验alert是否成功绑定

同时可查看邮箱是否有预警邮件

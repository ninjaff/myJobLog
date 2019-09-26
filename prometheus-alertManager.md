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
#### webhook
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
email
vim /usr/local/alertmanager/alertmanager.yml
```yaml
#全局配置,比如配置发件人
global:
  resolve_timeout: 5m    #处理超时时间，默认为5min
  smtp_smarthost: 'smtp.163.com:25'  # 邮箱smtp服务器代理
  smtp_from: 'zabbix@minminmsn.com' # 发送邮箱名称
  smtp_auth_username: 'zabbix@minminmsn.com' # 邮箱名称
  smtp_auth_password: '12345678xxOO'              # 邮箱密码或授权码

# 定义模板信息,可以自定义 html 模板,发邮件的时候用自己定义的模板内容发
templates:
  - 'template/*.tmpl'

# 定义路由树信息,这个路由可以接收到所有的告警,还可以继续配置路由,比如project: zhidaoAPP(prometheus 告警规则中自定义的lable)发给谁,project: baoxian的发给谁
route:
  group_by: ['alertname'] # 报警分组依据
  group_wait: 10s         # 最初即第一次等待多久时间发送一组警报的通知
  group_interval: 60s     # 在发送新警报前的等待时间
  repeat_interval: 1h     # 发送重复警报的周期 对于email配置中，此项不可以设置过低，否则将会由于邮件发送太多频繁，被smtp服务器拒绝
  receiver: 'email'       # 发送警报的接收者的名称，以下receivers name的名称

# 定义警报接收者信息
receivers:
  - name: 'email'  # 路由中对应的receiver名称
    email_configs: # 邮箱配置
    - to: 'admin@minminmsn.com'   # 接收警报的email配置
      #html: '{{ template "test.html" . }}'  # 设定邮箱的内容模板
```
run-alertManager.sh
```bash
#/bin/sh
pid_file=`dirname $0`/alertmanager.pid  
if [ -f "$pid_file" ] ; then  
    echo "AlertManager is still running, please stop first."
    exit
fi

nohup ./alertmanager --config.file=alertmanager.yml > alertmanager.log 2>&1 &  
echo $! > alertmanager.pid
```

stop-alertManager.sh
```bash
pidfile=`dirname $0`/alertmanager.pid  
if [ ! -f "$pidfile" ] ; then  
    echo "AlertManager Already Stopped."
    exit
fi

pid=`cat $pidfile`  
echo -e "`hostname` Stopping AlertManager $pid ... "  
kill $pid

if [ -f "$pidfile" ] ; then  
    rm $pidfile
    exit
fi
```

### test
通过kill被监控端的node_exporter进程可以试验alert是否成功绑定

同时可查看邮箱是否有预警邮件

### yaml检验工具
Prometheus 提供了验证规则文件的工具：

```./promtool check rules rules/node.yml```

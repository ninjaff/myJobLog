---
title: prometheus-alertManager
date: 2019-08-28 08:20:00
tags: [prometheus, alertManager]
---

### info
https://prometheus.io/docs/alerting/configuration/

https://github.com/prometheus/alertmanager

https://prometheus.io/download/

https://github.com/prometheus/alertmanager/releases/download/v0.19.0/alertmanager-0.19.0.linux-amd64.tar.gz

### deploy
```bash
tar xf alertmanager-x.x.x.linux-amd64.tar.gz
cd alertmanager-x.x.x.linux-amd64
./alertmanager --config.file=alertmanager.yml
```

### configer rules
vi /etc/prometheus/alert.rules
```yaml
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
```yaml
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
alertmanager configuration提供多种告警处理方式
```
email_config
hipchat_config
pagerduty_config
pushover_config
slack_config
opsgenie_config
victorops_config
webhook_config
wechat_config
```
#### alertmanager
```yaml
global:
  resolve_timeout: 5m
 
route:
  group_by: ['alertname']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 1h
  receiver: 'web.hook'
receivers:
- name: 'web.hook'
  webhook_configs:
  - url: 'http://127.0.0.1:5001/'
inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'dev', 'instance']
```

#### webhook
```yaml
route:
  group_by: ['alertname']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 1h
  receiver: 'web.hook'
receivers:
  - name: 'web.hook'
    webhook_configs:
    - url: 'https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=xxxx'
inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'dev', 'instance']
```
#### email
```yaml
#全局配置,比如配置发件人
global:
  resolve_timeout: 5m    #处理超时时间，默认为5min
#  smtp_smarthost: 'smtp.163.com:25'  # 邮箱smtp服务器代理
#  smtp_from: 'zabbix@minminmsn.com' # 发送邮箱名称
#  smtp_auth_username: 'zabbix@minminmsn.com' # 邮箱名称
#  smtp_auth_password: '12345678xxOO'              # 邮箱密码或授权码

# 定义模板信息,可以自定义 html 模板,发邮件的时候用自己定义的模板内容发
#templates:
#  - 'template/*.tmpl'

# 定义路由树信息,这个路由可以接收到所有的告警,还可以继续配置路由,比如project: zhidaoAPP(prometheus 告警规则中自定义的lable)发给谁,project: baoxian的发给谁
route:
  group_by: ['alertname'] # 报警分组依据
  group_wait: 10s         # 最初即第一次等待多久时间发送一组警报的通知
  group_interval: 10s     # 在发送新警报前的等待时间
  repeat_interval: 1h     # 发送重复警报的周期 对于email配置中，此项不可以设置过低，否则将会由于邮件发送太多频繁，被smtp服务器拒绝
  receiver: 'email'       # 发送警报的接收者的名称，以下receivers name的名称

# 定义警报接收者信息
receivers:
  - name: 'web.hook'
    webhook_configs:
    - url: 'http://127.0.0.1:5001/'
  - name: 'email'  # 路由中对应的receiver名称
    email_configs: # 邮箱配置
    - to: 'admin@minminmsn.com'   # 接收警报的email配置
      #html: '{{ template "test.html" . }}'  # 设定邮箱的内容模板
      from: 你的发件用的网易邮箱
      smarthost:  smtp.163.com:25
      auth_username: 网易邮箱账号
      auth_password: 网易邮箱密码
      # auth_secret:
      # auth_identity:
inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'dev', 'instance']
```
#### wechat
```yaml
route:
  group_by: ['...']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 10m  
  receiver: 'wechat'
templates:
  - '/server/alertmanager/template/wechat.tmpl'
receivers:
- name: 'wechat'
  wechat_configs:
  - corp_id: 'wwef5c3f7b19a22cc5'
#    send_resolved: true
#    to_party: '2'
#    to_user: 'ShenYaMing'
    agent_id: '1000002'
    api_secret: 'ZlZD7peO5LC3_9tWnins-Kys8AS0PaiGzgzc3amcc24'
```
##### wechat.tmpl
```
{{- define "__text_alert_list" -}}
{{- range .Alerts.Firing -}}
信息: [惊恐][惊恐][惊恐]
级别: {{ .Labels.severity }}
时间: {{ .StartsAt.Format "2006-01-02 15:04:05" }}
描述: {{ .Annotations.description }}
{{- end }}
{{- range .Alerts.Resolved -}}
信息: [悠闲][悠闲][悠闲]
级别: {{ .Labels.severity }}
时间: {{ .StartsAt.Format "2006-01-02 15:04:05" }}
描述: {{ .Annotations.description }}
{{- end }}
{{- end }}


{{- define "wechat.default.message" -}}
{{- if gt (len .Alerts.Firing) 0 -}}
{{ template "__text_alert_list" . }}
{{- end }}
{{- if gt (len .Alerts.Resolved) 0 -}}
{{ template "__text_alert_list" . }}
{{- end }}
{{- end }}
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

### alertmanager集群模式
alertmanager可以配置成集群模式，即多个alaertmanager一起运行，彼此之间通过gossip协议获知告警的处理状态，防止告警重复发出。

这种模式通常用在prometheus需要做高可用的场景中。

prometheus ha deploy的高可用部署通常至少会有两套prometheus独立工作，它们会执行各自的告警检查。

与之相伴的通常也要部署多个alaertmanager，这时候这些alertmanager之间就需要同步信息，防止告警重复发出。

由于使用的是gossip协议，alermanager的集群模式配置很简单，只需要启动时指定另一个或多个alertmanager的地址即可：

`--cluster.peer=192.168.88.10:9094`


### on k8s ConfigMap
alertmanager-conf.yaml
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: alert-config  
  namespace: kube-ops
data:
  config.yml: |-
    global:
      # 在没有报警的情况下声明为已解决的时间
      resolve_timeout: 5m      # 配置邮件发送信息
      smtp_smarthost: 'smtp.163.com:25'
      smtp_from: 'ych_1024@163.com'
      smtp_auth_username: 'ych_1024@163.com'
      smtp_auth_password: '<邮箱密码>'
      smtp_hello: '163.com'
      smtp_require_tls: false
    # 所有报警信息进入后的根路由，用来设置报警的分发策略
    route:
      # 这里的标签列表是接收到报警信息后的重新分组标签，例如，接收到的报警信息里面有许多具有 cluster=A 和 alertname=LatncyHigh 这样的标签的报警信息将会批量被聚合到一个分组里面
      group_by: ['alertname', 'cluster']
      # 当一个新的报警分组被创建后，需要等待至少group_wait时间来初始化通知，这种方式可以确保您能有足够的时间为同一分组来获取多个警报，然后一起触发这个报警信息。
      group_wait: 30s      # 当第一个报警发送后，等待'group_interval'时间来发送新的一组报警信息。
      group_interval: 5m      # 如果一个报警信息已经发送成功了，等待'repeat_interval'时间来重新发送他们
      repeat_interval: 5m      # 默认的receiver：如果一个报警没有被一个route匹配，则发送给默认的接收器
      receiver: default      # 上面所有的属性都由所有子路由继承，并且可以在每个子路由上进行覆盖。
      routes:
      - receiver: email        
        group_wait: 10s        
        match:
          team: node    
    receivers:
    - name: 'default'
      email_configs:
      - to: '517554016@qq.com'
        send_resolved: true
    - name: 'email'
      email_configs:
      - to: '517554016@qq.com'
        send_resolved: true
```

创建 ConfigMap 资源对象
kubectl create -f alertmanager-conf.yaml

prome-deploy.yaml
```yaml
 - name: alertmanager    
    image: prom/alertmanager:v0.15.3    
    imagePullPolicy: IfNotPresent    
    args:
    - "--config.file=/etc/alertmanager/config.yml"
    - "--storage.path=/alertmanager/data"
    ports:
    - containerPort: 9093
      name: http    
    volumeMounts:
    - mountPath: "/etc/alertmanager"
      name: alertcfg    
    resources:
      requests:
        cpu: 100m        
        memory: 256Mi      
      limits:
        cpu: 100m        
        memory: 256Mi
volumes:
- name: alertcfg  
  configMap:
    name: alert-config
```
kubectl apply -f prome-deploy.yaml

Prometheus中配置AlertManager
prome-cm.yaml
```yaml
alerting:
  alertmanagers:
    - static_configs:
      - targets: ["localhost:9093"]
```
kubectl delete -f prome-cm.yaml
kubectl create -f prome-cm.yaml

kubectl get pods -n kube-ops

### test
通过kill被监控端的node_exporter进程可以试验alert是否成功绑定

同时可查看邮箱是否有预警邮件

### yaml检验工具
Prometheus 提供了验证规则文件的工具：

`./promtool check rules rules/node.yml`

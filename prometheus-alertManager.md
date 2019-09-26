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
tar xf alertmanager-x.x.x.linux-amd64.tar.gz
cd alertmanager-x.x.x.linux-amd64
./alertmanager --config.file=alertmanager.yml

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
```yaml
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

---
title: mysql-exporter.md
date: 2019-08-26 18:18:20
tags: [prometheus, exporter, mysql]
---

### download
https://github.com/prometheus/mysqld_exporter/releases/download/v0.12.1/mysqld_exporter-0.12.1.darwin-amd64.tar.gz

### edit prometheus.yml
```yaml
      - job_name: mysqldb1
        static_configs:
          - targets: ['test.mysql.com:9104']
            labels:
              instance: mysqldb1
	      group: staging
```
### add mysql user for exporter
```bash
root@master:mysql>GRANT REPLICATION CLIENT,PROCESS ON *.* TO 'prometheus'@'%' identified by '123456';
root@master:mysql>GRANT SELECT ON *.* TO 'prometheus'@'%';
```

### add exporter .my.cnf
```
$ cat > .my.cnf <<EOF
    [client]
    port=3306
    user=prometheus
    password=123456
EOF
```

### run exporter
```
./mysqld_exporter -config.my-cnf=”.my.cnf” &
## 后台启动
nohup 
```

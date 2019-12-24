# 安装部署
## yum方式
```
curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.4.2-x86_64.rpm
rpm -vi filebeat-7.4.2-x86_64.rpm
```
```
systemctl enable filebeat
```
配置
```
vim /etc/filebeat/filebeat.yml
```
```
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /data/logs/imp-register/imp-register.json
  tags: ["imp-register"]
  json.keys_under_root: true
  json.overwrite_keys: true
  json.message_key: message
filebeat.config.modules:
  path: ${path.config}/modules.d/*.yml
  reload.enabled: false
setup.template.name: "imp-register"
setup.template.pattern: "imp-register-*"
setup.ilm.enabled: false
output.elasticsearch:
  hosts: ["http://10.43.81.51:9200", "http://10.43.81.50:9200", "http://10.43.81.60:9200"]
processors:
  - drop_fields:
      fields: ["input","log","beat","offset","source","host","span","trace","parent"]
```
启动
```
systemctl start filebeat
```

# 配置使用
## 搜集json格式的日志
`/etc/filebeat/filebeat.yml`
```
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /data/logs/imp-register/imp-register-10.43.81.60.json
  fields:
    index: 'imp-register'
  json.keys_under_root: true
  json.overwrite_keys: true
  json.message_key: message
- type: log
  enabled: true
  paths:
    - /data/jars/imp_overedge_deduplicate/logs/imp_overedge_deduplicate/*.json
  fields:
    index: 'imp_overedge_deduplicate'
  json.keys_under_root: true
  json.overwrite_keys: true
  json.message_key: message
filebeat.config.modules:
  path: ${path.config}/modules.d/*.yml
  reload.enabled: false
output.elasticsearch:
  hosts: ["http://10.43.81.51:9200", "http://10.43.81.50:9200", "http://10.43.81.60:9200"]
  indices:
    - index: "imp-register-%{+yyyy.MM.dd}"
      when.contains:
        fields:
          index: 'imp-register'
    - index: "imp_overedge_deduplicate-%{+yyyy.MM.dd}"
      when.contains:
        fields:
          index: 'imp_overedge_deduplicate'
processors:
  - drop_fields:
      fields: ["input","log","beat","offset","source","host","span","trace","parent"]
```
日志格式示例
```
{"@timestamp":"2019-12-15T10:54:06.136Z","ip":"10.43.81.60","app":"imp-register","level":"INFO","trace":"","span":"","parent":"","thread":"http-nio-13001-exec-8","class":"c.e.imp.register.listener.InstanceRenewListener","line_number":"54","message":"appHeartInfo: AppHeartbeatInfo(appName=daa-imgpro-service, ip=172.30.128.62, port=15031, renewTime=Sun Dec 15 18:54:06 CST 2019)","stack_trace":""}
```
# kubernetes-elk
## kubernetes中使用elk收集容器日志

### 环境：
    docker 17.10.0
    filebeat 5.3.0
    logstash 5.3.2
    elasticsearch 2.3.4
    kibana 4.5.3

### 概述
在kubernetes集群中，容器所运行的node节点是由kubernetes scheduler 分配的，每个容器运行的节点都是不固定的。本处利用docker会将前端日志输出到/var/lib/docker/containers目录中，filebeat监听该目录，在filebeat中使用正则表达式区分不同的日志发往logstash，在logstash针对不同的日志，打上不同的index，最终发送到数据库中（elasticsearch）

### 前提条件：
### docker容器中服务产生的日志都导入该目录下/var/lib/docker/containers

### tips：
本处不涉及到kubernetes、elasticsearch、kibana安装配置，只涉及到如何收集docker容器日志以及分类发往数据库，并在kibana中展示出来

由于filebeat配置文件内容较多，故本处展示使用正则表达式分类的配置，完整配置文件https://github.com/samlyhuang/kubernetes-elk/blob/master/filebeat.yml：

    - input_type: log
    
      paths:
        - /tmp/log/nginx-html-error.log
      document_type: nginx-html-error-log
    
      include_lines: ['.*(?:[0-9]{1,4}\/[0-9]{1,2}\/[0-9]{1,2})\s[0-9]{1,2}\:[0-9]{1,2}\:[0-9]{1,2}\s\[error\]\s[0-9]{1,9}#.*']



logstash配置文件如下：

    input {
        beats {
            port => 5045
            }
    }
    
    output {
        if [type] == "nginx-access-log" {
            elasticsearch {
                hosts => ["192.168.31.153:9200"]
                index => "nginx-access-log-%{+YYYY.MM.dd}"
                flush_size => 20000
                idle_flush_time => 10
                template_overwrite => true
            }
        }
    
        if [type] == "nginx-html-error-log" {
            elasticsearch {
                hosts => ["192.168.31.153:9200"]
                index => "nginx-html-error-log-%{+YYYY.MM.dd}"
                flush_size => 20000
                idle_flush_time => 10
                template_overwrite => true
            }
        }
    }
    

在kibana页面中，使用“nginx-html-error-log”建立索引，即可看到docker容器中的日志。可能有一定的延迟，我这边大约延迟十多秒。



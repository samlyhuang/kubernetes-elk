# kubernetes-elk
## kubernetes中使用elk收集容器日志

### 前提条件：
### docker中服务产生的日志都通过容器导入该目录下/var/lib/docker/containers

在使用elk收集容器日志时，由于所有日志文件都存在于/var/lib/docker/containers目录下，此时主要问题就是如何把不同的日志数据进行分类。本处使用到filebeat工具中的正则表达式对日志进行分类，然后统一发送到logstash服务，在logstash中，根据不同的type，打上不同的index。

    由于filebeat配置文件内容较多，故本处展示使用正则表达式分类的配置，完整配置文件请查看根目录：
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

















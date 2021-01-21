# Springboot-ELK快速入门
Elasticsearch, Logstash, and Kibana Hello World Example 本例使用的版本号统一为：6.2.2

## 安装Elasticsearch
地址： (https://www.elastic.co/downloads/elasticsearch)

windows 运行方法： 
>点击 elasticsearch.bat

启动成功后，浏览器输入localhost:9200，返回如下代码说明启动成功： 
```
{
    "name": "_UwRJxM",  
    "cluster_name": "elasticsearch",  
    "cluster_uuid": "DfWELZcWRuWggCxtaee1YA",  
    "version": {  
        "number": "6.2.2",  
        "build_hash": "10b1edd",  
        "build_date": "2018-02-16T19:01:30.685723Z",  
        "build_snapshot": false,  
        "lucene_version": "7.2.1",  
        "minimum_wire_compatibility_version": "5.6.0",  
        "minimum_index_compatibility_version": "5.0.0"  
    },  
    "tagline": "You Know, for Search"  
}
```


## 安装Kibana
地址： (https://www.elastic.co/downloads/kibana)

修改kibana.yml配置文件，新增如下配置：

`elasticsearch.url: "http://localhost:9200"`

windows 运行方法： 

>点击 kibana.bat

启动成功后，浏览器输入localhost:5601，访问Kibana UI页面，页面样式如下：

![Image text](https://ss1.bdstatic.com/70cFvXSh_Q1YnxGkpoWK1HF6hhy/it/u=1228117569,2186082610&fm=26&gp=0.jpg)





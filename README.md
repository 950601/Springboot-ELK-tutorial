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


## 安装logstash
地址： (https://www.elastic.co/downloads/logstash)

创建conf/logstash.conf
```
input {
  file {
    type => "java"
    path => "C:/elk/spring-boot-elk.log"
    codec => multiline {
      pattern => "^%{YEAR}-%{MONTHNUM}-%{MONTHDAY} %{TIME}.*"
      negate => "true"
      what => "previous"  
    }
  } 
}
 
filter {
  #If log line contains tab character followed by 'at' then we will tag that entry as stacktrace
  if [message] =~ "\tat" {
    grok {
      match => ["message", "^(\tat)"]
      add_tag => ["stacktrace"]
    }
  }
 
}
 
output {
   
  stdout { 
    codec => rubydebug
  }
 
  # Sending properly parsed log events to elasticsearch
  elasticsearch {
    hosts => ["localhost:9200"]
  }
}
```

进入安装bin目录，输入如下命令，启动logstash服务

>logstash.bat -f logstash.conf

## 创建springboot项目，用来生成日志

定义pom.xml如下：

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.javainuse</groupId>
	<artifactId>spring-boot-elk</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>

	<name>spring-boot-elk</name>

	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.5.6.RELEASE</version>
		<relativePath /> <!-- lookup parent from repository -->
	</parent>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
		<java.version>1.8</java.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>


</project>
```

编写controller，暴露日志生成接口，我们将利用这个接口生成异常日志

```
import java.io.PrintWriter;
import java.io.StringWriter;
import java.util.Date;

import org.apache.log4j.Level;
import org.apache.log4j.Logger;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

@RestController
class ELKController {
	private static final Logger LOG = Logger.getLogger(ELKController.class.getName());

	@Autowired
	RestTemplate restTemplete;

	@Bean
	RestTemplate restTemplate() {
		return new RestTemplate();
	}

	@RequestMapping(value = "/elk")
	public String helloWorld() {
		String response = "Welcome to JavaInUse" + new Date();
		LOG.log(Level.INFO, response);

		return response;
	}

	@RequestMapping(value = "/exception")
	public String exception() {
		String response = "";
		try {
			throw new Exception("Exception has occured....");
		} catch (Exception e) {
			e.printStackTrace();
			LOG.error(e);

			StringWriter sw = new StringWriter();
			PrintWriter pw = new PrintWriter(sw);
			e.printStackTrace(pw);
			String stackTrace = sw.toString();
			LOG.error("Exception - " + stackTrace);
			response = stackTrace;
		}

		return response;
	}
}
```

修改application.properties或 application.yml, 添加日志文件的位置 

```
logging.file=C:/elk/spring-boot-elk.log
```

启动项目，访问日志生成接口，日志将被生成在 C:/elk/spring-boot-elk.log

>localhost:8080/elk

>localhost:8080/exception

## 访问 kibana-ui (http://localhost:5601/) 

- 定义索引规则，菜单入口：

- management -> index pattern -> create index pattern 

- 输入 logstash-*，接下来便可在kibana查看logstash输出的日志

**结束**












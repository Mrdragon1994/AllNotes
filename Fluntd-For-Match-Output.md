---
title: Fluntd For Match(Output)
date: 2020-03-25 22:20:12
tags:
- Fluentd
- Match
categories:
- ELK
- 运维
- Fluentd
---

# \<match\>输出

- 可以被输出的位置也都多种多样的，目前支持file，forward，http，exec，exec_filter，secondary_file，copy，stdout，s3，kafka，elasticsearch，mongo…

- 由于目前搭建的环境是ELK，是直接吐向elasticsearch的，因此目前学习的@Type就是elasticsearch

- @Type参数  //必须为：elasticsearch

- host属性 //可选，ES的主机名，可以配置为实际的IP地址，默认是localhost；

- port属性 //可选，ES的端口号，默认9200

- hosts属性  //可选，如果ES为集群模式，那么可以使用此参数，配置如下：

  ````json
  hosts host1:port1,host2:port2,host3:port3
  # or
  hosts https://customhost.com:443/path,https://username:password@host-failover.com:443
  ````

  如果使用了hosts，那么host和port就失效了。

- user / password属性 //登录ES节点的用户名和密码，默认是nil

- schema属性  //默认是http,如果ES支持SSL，可以更改为https

- path属性  //ES发布写请求的REST API

- logstash_format属性 //默认是false，如果改为了true，那么就会使用logstash规范建立index

- logstach_prefix属性 //默认情况下建立索引是logstash_%Y.%m.%d，但是配置了这个属性后，便会使用自己的配置将logstash更改掉，如使用${tag}

- include_timestamp属性  //默认是false，改为true，便会在日志上一个@timestamp

- ssl_version TLSv1_2

- ssl_verify属性  //默认是true，可改为false跳过SSL检测


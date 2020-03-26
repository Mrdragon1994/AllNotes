---
title: Fluntd For Source(Input)
date: 2020-03-25 22:20:02
tags:
- Fluentd
- Source(input/tail)
categories:
- ELK
- 运维
- Fluentd
---

# \<source\>

- source有很多种类，tail,forward,udp,tcp,unix,http,syslog,exec,dummy,monitor_agetn,windows_eventlog；

- 我们目前只用来检测和梳理日志，因此选用tail即可；

- 前提举例：

  ```xml
  <source>
    @type tail
    path /var/log/httpd-access.log
    pos_file /var/log/td-agent/httpd-access.log.pos
    tag apache.access
    <parse>
      @type apache2
    </parse>
  </source>
  ```

- @Type 必填项 //目前只用来收集日志，因此我们只写为tail即可；

- tag属性  //必备属性,当前source的标记,可以自己手动输出,也可以使用*来充当占位符

  ```json
  path /path/to/file
  tag foo.*
  ```

  如上输入，则最后的tag为**foo.path.to.file**;

- path属性  //必备属性,指定需要读取的日志位置，可以使用\*号来正则匹配路径，若有多个路径则可以用逗号分隔开；

  ```json
  path /path/to/%Y/%m/%d/*
  path /path/to/a/*,/path/to/b/c.log
  ```

- path_timezone属性  //非必须，默认使用系统时区，主要是针对日志路径为: /path/%Y/%m/%d的；

  ```json
  path_timezone "+00"
  ```

  

- exclude_path属性  //非必须，默认是空数组，主要是移除部分不需要采集的日志

  ```json
  path /path/to/*
  exclude_path ["/path/to/*.gz", "/path/to/*.zip"]
  ```

- refresh_interval属性 //默认值是60s，用来控制刷新文件列表的间隔，当路径中含有\*时，该值一定要使用，但是可以不写，因为有默认值;

- skip_refresh_on_startup属性  //当重启时跳过刷新文件列表，可以大幅减少路径中包含\*的path的时间;

- read_from_head属性 //当重启时，将会从头开始读取日志文件,默认值是false,建议改为true

- read_lines_limit属性 //每次IO读取的文件行数，默认是1000，**如果看到chunk bytes limit exceeds for an emitted event stream**，需要将该值设小;

- multiline_flush_interval属性 //在\<parse\>为multiline Mode下，该值很有用，设置如下：

  ```json
  multiline_flush_interval 5s
  ```

- **pos_file** //虽然为没有，但是一定要设置,因为当我们重启后，他要去找到当前读到哪里了,这个路径不要和path重复；

- pos_file_compaction_interval属性 //合并压缩pos_file文件，当pos_file开启时，建议使用 ：

  ```json
  pos_file /var/log/td-agent/tmp/access.log.pos
  pos_file_compaction_interval 72h
  ```

- \<parse\>标签 //如果source的@Type为tail,则该标签为必需！！！该标签下的属性和@Type见：[parse](http://39.100.22.86/2020/03/25/Fluentd-For-Parse/)


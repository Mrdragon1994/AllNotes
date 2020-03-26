---
title: Fluentd For Buffer
date: 2020-03-26 10:23:54
tags:
- Fluentd
- Buffer
categories:
- ELK
- 运维
- Fluentd
---

# \<buffer\>

- 首先需要明确的是,\<buffer\>是用在输出插件中的，也就是用在\<match\>中的；

- 设置缓存区的目的是控制重试行为，当一个缓存块因各种原因没有将数据送到目的地时，便可以进行重试；

- buffer的参数可以是：

  - @Type file //内置类型
  - @Type memory  //内置类型
  - @Type file_single  //第三方

- 需要注意的是，\<buffer\>的@Type是可以省略的,默认是memory，但是建议配置该属性

- buffer的几种Chunk keys

  - Blank_chunk_keys //默认配置的是该值，它会将读到的所有信息全部写到一个chunk中，直到达到指定的大小

    ```xml
    <buffer>
    	...
    </buffer>
    ```

  - Tag  //拥有不同tag的event会被写入到不同的chunks中

    ```xml
    <buffer tag>
        # ...
    </buffer>
    ```

  - Time  //依据timekey设定的时间来进行chunk的划分，其中有2个属性，timekey和timekey_wait

    ```xml
    <buffer time>
        timekey      1h # chunks per hours ("3600" also available)
        timekey_wait 5m # 5mins delay for flush ("300" also available)
      </buffer>
    ```

    ```xml
    <match tag.**>
      # ...
      <buffer time>
        timekey      1h # chunks per hours ("3600" also available)
        timekey_wait 5m # 5mins delay for flush ("300" also available)
      </buffer>
    </match>
    
    # Time chunk key: events will be separated for hours (by timekey 3600)
    
    11:59:30 web.access {"key1":"yay","key2":100}  ------> CHUNK_A
    
    12:00:01 web.access {"key1":"foo","key2":200}  --|
                                                     |---> CHUNK_B
    12:00:25 ssh.login  {"key1":"yay","key2":100}  --|
    ```

    也就是11:00:00到11:59:59秒都会属于chunk-A

    12:00:00–12:59:59就会属于chunk-B

    timekey_wait的值会延迟chunk实际刷新时间：

    ```json
     timekey: 3600
     -------------------------------------------------------
     time range for chunk | timekey_wait | actual flush time
      12:00:00 - 12:59:59 |           0s |          13:00:00
      12:00:00 - 12:59:59 |     60s (1m) |          13:01:00
      12:00:00 - 12:59:59 |   600s (10m) |          13:10:00
    ```

  - 还有其他几种chunk的选择，由于不常用，暂不学习了，详见：https://docs.fluentd.org/configuration/buffer-section

- \<buffer\>的chunk是可以配置成多个的

  ```xml
  <buffer>
    # ...
  </buffer>
  
  <buffer tag, time, key1>
    # ...
  </buffer>
  ```

  

- @Type属性 //配置的缓存存放区

  ---time设置

- 如果指定的chunk的类型为time，那么就有几个和time有关的优化参数：

  - timekey属性 //必须参数,控制一个时间范围去刷新chunk
  - timekey_wati属性 //默认是600（10m）,该值会影响timekey的实际刷新时间；
  - timekey_use_utc 属性 //默认是false，使用本地时区
  - timekey_zone属性 //默认使用本地时区,可以设置 -0700 or Asia/Tokyo

  --- buffer设置

- chunk_limit_size属性 //默认是8MB（memory）或者256MB（file），该值控制了每个chunk的大小

- chunk_limit_records属性 //每个chunk中可以存的最大的events数

- total_limit_size属性  //默认512MB（memory），64GB（file），缓存区的大小，一旦达到这个阈值，所有的追加操作都会失败，也就意味着所有的数据都会丢失

- chunk_full_threshold属性 //默认0.95，虽然chunk_limit_size控制了chunk的大小，但是并不是等于该大小时才会写入到下一个chunk中，而是有一个阈值，即当chunk_limit_size * chunk_full_threshold （== 8MB * 0.95） 达到该值时，便会写入到下一个chunk中

- compress属性  //chunk是否开启压缩，默认是text（不压缩），压缩是需要设置为gzip

  === flush设置

- flush_at_shutdown属性 //对于file类型默认是false，对于memory是true类型

- flush_mode属性 //刷新模式，有default，lazy，interval,immediate

  如果\<buffer\>设置了time类型，那么default就是lazy，否则就是interval

  lazy：每个timekey才刷新一次

  interval：通过flush_interval来控制时间周期

  immediate·：当收到一条数据后就立刻刷新

- flush_interval属性  //默认60s，配置flush_mode为interval时使用

- flush_thread_count  //默认线程时1，可以设置多个，让chunk并发运行

- flush_thread_interval属性 //当没有chunk在等待时，线程的休眠周期，默认1.0

- flush_thread_burst_interval属性 //当chunk有等待时，线程的休眠周期，默认是1.0

- delayed_commit_timeout属性 //默认是60，当输出线程感知异步write操作失败的超时时间

- overflow_action属性 //当buffer queue满时，该怎么处理, 

  throw_exception  //默认是抛出异常

  block  //阻塞输入的进程，以免继续向buffer中发送数据

  drop_oldest_chunk属性 //丢弃老的chunk，以便接收新的chunk

  === retry设置

- retry_timeout属性 //默认72h，当刷新chunk失败时，最长的重试时间

- retry_forever属性 //默认是false，如果设置为true，那么就会忽略**retry_timeout**和**retry_max_times**的配置，而永远去重试

- retry_max_times属性 //默认是none，当失败时最大的重试次数

- retry_secondary_threshold属性 //默认0.8，最大是1.0，当失败超时达到这个阈值时，启用第二配置

- retry_wait 属性 //默认是1s,在下一个重试刷新之前的等待时间

  

- 当@Type是file的时候

  - path   //必备,如果写了\*，那么会被任意字符给替换掉，且要求改路径一定是唯一的，不歧义的；
  - path_suffix   //默认是.log

  








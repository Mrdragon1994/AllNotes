---
title: Fluentd For Parse
date: 2020-03-25 21:48:06
tags:
- Fluentd
- Parse
categories:
- ELK
- 运维
- Fluentd
---

# \<parse\>

首先需要注意的是：\<parse\>可以用在\<source\>,\<filter\>,\<match\>中。

支持\<parse\>的输入插件(即在\<source\>中的@Type为下面几个时)有：tail, tcp, udp, syslog,http。

由于目前只对日志进行捕获，因此我们的输入插件只用tail即可。

- 先介绍一些写在\<parse\>里面的属性

  - @Type  //表示当前的parse解析插件需要将日志解析成何种格式，目前常用到的有regexp，nginx，csv，json，multiline；

  - types属性  //当我们从日志中解析出部分字段时,可以指定某些字段的数据类型,目前支持string,bool,integer,float,time,array

    ```json
    types user_id:integer,paid:bool,paid_usd_amount:float
    ```

    

  - time_key属性  //当我们捕获的正则组有命名为time的，那么可以使用time_key来指定它，并且可以使用time_format来转换格式

  - estimate_current_event属性  //当time_key指定后，该属性会使用F`luent::EventTime.now`(current time)作为timestamp，该值默认是**true**

  - keep_time_key属性   //默认是**false**，设置为true，将保证time字段存在于record中；

  - timeout 属性  //默认没有，这是为了方式当有错误的正则表达式

  - time的一些特性

    - time默认是string，还有float，unixtime

      当为string时，则可以按照time_format或者当地时间或者时区进行格式的输出；

      当为float时，就会输出浮点型的时间戳；

      当为unixtime时，便会输出整形的时间戳；

      ```json
      types date:time:%d/%b/%Y:%H:%M:%S %z # for string with time format
      types date:time:unixtime             # for integer time
      types date:time:float                # for float time
      ```

      字段:time:格式

    - time_format属性 //仅当time_type为string类型时，该属性才生效,因次想用的话，直接写为：

      ```json
      types date:time:%d/%b/%Y:%H:%M:%S %z # for string with time format
      ```
      
      格式有：
      
      %a
      
      The abbreviated weekday name (“Sun”)
      
      %A
      
      The full weekday name (“Sunday”)
      
      %b
      
      The abbreviated month name (“Jan”)
      
      %B
      
      The full month name (“January”)
      
      %c
      
      The preferred local date and time representation
      
      %C
      
      Century (20 in 2009)
      
      %d
      
      Day of the month (01..31)
      
      %D
      
      [`Date`](https://docs.ruby-lang.org/en/2.4.0/Date.html) (%m/%d/%y)
      
      %e
      
      Day of the month, blank-padded ( 1..31)
      
      %F
      
      Equivalent to %Y-%m-%d (the ISO 8601 date format)
      
      %h
      
      Equivalent to %b
      
      %H
      
      Hour of the day, 24-hour clock (00..23)
      
      %I
      
      Hour of the day, 12-hour clock (01..12)
      
      %j
      
      Day of the year (001..366)
      
      %k
      
      hour, 24-hour clock, blank-padded ( 0..23)
      
      %l
      
      hour, 12-hour clock, blank-padded ( 0..12)
      
      %L
      
      Millisecond of the second (000..999)
      
      %m
      
      Month of the year (01..12)
      
      %M
      
      Minute of the hour (00..59)
      
      %n
      
      Newline (n)
      
      %N
      
      Fractional seconds digits, default is 9 digits (nanosecond)
      
      - %3N
      
        millisecond (3 digits)
      
      - %6N
      
        microsecond (6 digits)
      
      - %9N
      
        nanosecond (9 digits)
      
      %p
      
      Meridian indicator (“AM” or “PM”)
      
      %P
      
      Meridian indicator (“am” or “pm”)
      
      %r
      
      time, 12-hour (same as %I:%M:%S %p)
      
      %R
      
      time, 24-hour (%H:%M)
      
      %s
      
      Number of seconds since 1970-01-01 00:00:00 UTC.
      
      %S
      
      Second of the minute (00..60)
      
      %t
      
      Tab character (t)
      
      %T
      
      time, 24-hour (%H:%M:%S)
      
      %u
      
      Day of the week as a decimal, Monday being 1. (1..7)
      
      %U
      
      Week number of the current year, starting with the first Sunday as the first day of the first week (00..53)
      
      %v
      
      VMS date (%e-%b-%Y)
      
      %V
      
      Week number of year according to ISO 8601 (01..53)
      
      %W
      
      Week number of the current year, starting with the first Monday as the first day of the first week (00..53)
      
      %w
      
      Day of the week (Sunday is 0, 0..6)
      
      %x
      
      Preferred representation for the date alone, no time
      
      %X
      
      Preferred representation for the time alone, no date
      
      %y
      
      Year without a century (00..99)
      
      %Y
      
      Year which may include century, if provided
      
      %z
      
      [`Time`](https://docs.ruby-lang.org/en/2.4.0/Time.html) zone as hour offset from UTC (e.g. +0900)
      
      %Z
      
      [`Time`](https://docs.ruby-lang.org/en/2.4.0/Time.html) zone name
      
      %%
      
      Literal “%” character



- 介绍一下\<parse\>的不同@Type的特色

  - regexp

    1. 需要使用?\<name\>这样的捕获组，如果捕获组命名为time，那么可以使用time_key对其进行指向，并且可以使用time_format进行格式转换。

       格式为：	

       ```xml
       <parse>
         @type regexp
         expression /.../
       </parse>
       ```

    2. 正则表达式支持i和m的结尾写法

       当为i时：表明ignorecase，忽略大小写

       当为m时，表明开启multiline，此时就相当于@Type为multiline mode

       也可以两者同时使用

       ```json
       expression /.../i
       expression /.../m
       expression /.../im
       ```

       ```xml
       <parse>
         @type regexp
         expression /^\[(?<logtime>[^\]]*)\] (?<name>[^ ]*) (?<title>[^ ]*) (?<id>\d*)$/
         time_key logtime
         time_format %Y-%m-%d %H:%M:%S %z
         types id:integer
       </parse>
         
       //types logtime:time:%Y-%m-%d %H:%M:%S %z # for string with time format    或者这样写
       ```
  
  - json      
  
  - multiline
  
    1. 和正则模式差不多，是regexp的multiline模式；
    2. 需要使用format_firstline和formatN，N的范围是0-20








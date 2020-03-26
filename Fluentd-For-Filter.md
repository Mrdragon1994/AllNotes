---
title: Fluentd For Filter
date: 2020-03-25 21:46:47
tags:
- Fluentd
- Filter
categories:
- ELK
- 运维
- Fluentd
---

#  \<filter\>的字段说明

filter的作用是：

- 重新筛选出一些字段；
- 通过添加新的字段来丰富ES界面；
- 删除或屏蔽某些不宜展示的内容。

filter的几种type

- record_transformer

  这个插件应当被第一个使用

  ```xml
  <filter foo.bar>
    @type record_transformer
    <record>
      hostname "#{Socket.gethostname}"
      tag ${tag}  //这样表示总的tag, ${record["tag"]}，这样表示每条信息中的tag
    </record>
  </filter>
  ```

  ```json
  原本：
  {"message":"hello world!"}
  现在
  {"message":"hello world!", "hostname":"db001.internal.example.com", "tag":"foo.bar"}
  ```

  或者：

  ```xml
  <filter foo.bar>
    @type record_transformer
    enable_ruby
    <record>
      avg ${record["total"] / record["count"]}
    </record>
  </filter>
  ```

  ```json
  原本：
  {"total":100, "count":10}
  现在：
  {"total":100, "count":10, "avg":"10"}
  //请注意，在本例中，“avg”字段被键入为字符串。您可以使用auto_typecast trueoption将字段视为浮点数。
  ```

  如上，如果使用了**enable_ruby**，便可以在${…}中使用Ruby Expression了，

  修改已有属性：

  ```xml
  <filter foo.bar>
    @type record_transformer
    <record>
      message yay, ${record["message"]}
    </record>
  </filter>
  ```

  ```json
  原本：
  {"message":"hello world!"}
  现在：
  {"message":"yay, hello world!"}
  ```

  ```xml
  <filter web.*>
    @type record_transformer
    <record>
      service_name ${tag_parts[1]}
    </record>
  </filter>
  ```

  ```json
  假如tag为web.auth
  原本：
  {"user_id":1, "status":"ok"}
  现在：
  {"user_id":1, "status":"ok", "service_name":"auth"}
  ```

  @Type 必须为 **record_transformer**

  \<record\>指令：包含在\<record\>里面的东西会被认为是新的K-V对存入到ES中，

  e.g. 

  ```xml
  <record>
    NEW_FIELD NEW_VALUE
  </record>  
  ```

  当然我们可以使用${…}的语法来动态生成

  1. 在ES中已经解析出的字段可以被直接使用,{“total”:100} => record[“total”] = 100
  2. tag 表示tag全称
  3. `tag_parts[N]`refers to the Nth part of the tag.
  4. `tag_prefix[N]`refers to the [0..N] part of the tag.
  5. `tag_suffix[N]`refers to the [N..] part of the tag.

  All indices are zero-based. For example, if you have an incoming event tagged`debug.my.app`, then`tag_parts[1]`will represent "my". Also in this case,`tag_prefix[N]`and`tag_suffix[N]`will work as follows:

  ```xml
  tag_prefix[0] = debug          tag_suffix[0] = debug.my.app
  tag_prefix[1] = debug.my       tag_suffix[1] = my.app
  tag_prefix[2] = debug.my.app   tag_suffix[2] = app
  ```

  6. enable_ruby 属性 //默认是false，开启后便可以使用Ruby 的表达式了

  7. remove_keys //类型是array，表示移除的keys

  8. 嵌套字段

     ```json
     ${record.dig("top", "nest1", "nest2")}
     ```

- grep

  1. 含义是从已有的record中挑选新的字段加入到record中去，或者从record中移除

     ```xml
     <filter foo.bar>
       @type grep
       <regexp>
         key message
         pattern /cool/
       </regexp>
       <regexp>
         key hostname
         pattern /^web\d+\.example\.com$/
       </regexp>
       <exclude>
         key message
         pattern /uncool/
       </exclude>
     </filter>
     ```

     上面代码的含义是：

     1. “message”字段的值中包含“cool”的;
     2. “hostname”字段的值能够符合正则表达式：/^web\d+\.example\.com$/ 的
     3. “message”字段的值中不能包含“uncool”

     因此，以下信息会被保留：

     ```json
     {"message":"It's cool outside today", "hostname":"web001.example.com"}
     {"message":"That's not cool", "hostname":"web1337.example.com"}
     ```

     而以下信息会被抛弃：

     ```json
     {"message":"I am cool but you are uncool", "hostname":"db001.example.com"}   //因为message包含uncool
     {"hostname":"web001.example.com"} //因为没有message
     {"message":"It's cool outside today"} //因为没有hostname
     ```

  2. @Type 必须为grep

  3. \<and\>指令，就是同时包含\<regexp\>和\<exclude\>

     ```xml
     <and>
       <regexp>
         key price
         pattern /[1-9]\d*/
       </regexp>
       <regexp>
         key item_name
         pattern /^book_/
       </regexp>
     </and>
     ```

     等同于:

     ```xml
     <regexp>
       key price
       pattern /[1-9]\d*/
     </regexp>
     <regexp>
       key item_name
       pattern /^book_/
     </regexp>
     ```

     或者：

     ```xml
     <and>
       <exclude>
         key container_name
         pattern /^app\d{2}/
       </exclude>
       <exclude>
         key log_level
         pattern /^(?:debug|trace)$/
       </exclude>
     </and>
     ```

  4. \<or\>标签，

     ```xml
     <or>
       <exclude>
         key status_code
         pattern /^5\d\d$/
       </exclude>
       <exclude>
         key url
         pattern /\.css$/
       </exclude>
     </or>
     ```

     等同于：

     ```xml
     <exclude>
       key status_code
       pattern /^5\d\d$/
     </exclude>
     <exclude>
       key url
       pattern /\.css$/
     </exclude>
     ```

     或者：

     ```xml
     <or>
       <regexp>
         key container_name
         pattern /^db\d{2}/
       </regexp>
       <regexp>
         key log_level
         pattern /^(?:warn|error)$/
       </regexp>
     </or>
     ```

  5. \<regexp\>标签，指定过滤规则，它包含两个参数**key**,**pattern** 

     **key**是要应用正则表达式的那个字段名;

     **pattern**是正则表达式

     e.g

     ```xml
     <regexp>
       key price
       pattern /[1-9]\d*/
     </regexp>
     <regexp>
       key item_name
       pattern /^book_/
     </regexp>
     ```

     price必须是正数，item_name必须以book开头。

     ```xml
     <regexp>
       key item_name
       pattern /(^book_|^article)/
     </regexp>
     ```

     item_name需要以book或者article开头。

  6. \<exclude\>标签，移除那些符合正则表达式的,包含两个参数

   **key**

   **pattern**

 - parser 

   1. 使用key_name获取需要改变的字段名，然后对该字段的属性使用\<parse\>中的regexp,json,mutilint进行格式转换

   2. \<parse\>标签, 然后可以使用\<parse\>那一套@Type json, regexp, mutiline

   3. key_name属性 //必备属性,指定需要被转换的属性

   4. reverse_time属性 //在原始事件中保留时间，默认是false

   5. reverse_data 属性  //将原始JOSN格式的数据中的K_V提取出去，并且仍旧保留原始的JSON数据，默认是false

      ```xml
      <filter foo.bar>
        @type parser
        key_name log
        reserve_data true
        <parse>
          @type json
        </parse>
      </filter>
      ```

      ```json
      # input data:  {"key":"value","log":"{\"user\":1,\"num\":2}"}
      # output data: {"key":"value","log":"{\"user\":1,\"num\":2}","user":1,"num":2}
      ```

      如果设置为false，则就不会在保留原始的JSON串

      ```json
      # input data:  {"key":"value","log":"{\"user\":1,\"num\":2}"}
      # output data: {"user":1,"num":2}
      ```

   6. remove_key_name_field属性 //解析完移除key_name字段,默认是false

      ```xml
      <filter foo.bar>
        @type parser
        key_name log
        reserve_data true
        remove_key_name_field true
        <parse>
          @type json
        </parse>
      </filter>
      ```

      结果：

      ```json
      # input data:  {"key":"value","log":"{\"user\":1,\"num\":2}"}
      # output data: {"key":"value","user":1,"num":2}
      ```

      由于reverse_data,因此保留了key，又因为设置了remove_key_name_field，因此没有了log

   7. inject_key_prefix属性 //给解析出的增加前缀

      ```xml
      <filter foo.bar>
        @type parser
        key_name log
        reserve_data true
        inject_key_prefix data.
        <parse>
          @type json
        </parse>
      </filter>
      ```

      结果：

      ```json
      # input data:  {"log": "{\"user\":1,\"num\":2}"}
      # output data: {"log":"{\"user\":1,\"num\":2}","data.user":1, "data.num":2}
      ```

      